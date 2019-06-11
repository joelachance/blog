---
title: "How to use your Elasticache store on your local Redis Server 🎉"
description: "If you’re like me, you probably hate having to sync databases — especially if you’re using Elasticache since they currently have SYNC disabled. I found a couple of solutions, this one being the most…"
date: "2018-10-24T20:08:11.816Z"
categories: 
  - AWS
  - Bash Script
  - Elasticache

published: true
canonical_link: https://medium.com/joelachance/how-to-use-your-elasticache-store-on-your-local-redis-server-6a675eacfe9a
redirect_from:
  - /how-to-use-your-elasticache-store-on-your-local-redis-server-6a675eacfe9a
---

![](./asset-1.png)

If you’re like me, you probably hate having to sync databases — especially if you’re using Elasticache since they currently have `SYNC` disabled.

I found a couple of solutions, this one being the most promising: [https://github.com/stickermule/rump](https://github.com/stickermule/rump), however, it’s not available via a package manager, and I didn’t want to install Go on my computer 😖

Not having some of the common Redis commands available in Elasticache, I resorted to Redis dumps. This may or may not be a viable solution for you. Our shop currently has fairly small Redis stores (in the tens of MBs), so this script runs pretty quickly for us. If you have stores in the 100’s of MB’s, this will take a while to run.

A quick overview:

This script uses`aws-cli` to initiate an Elasticache backup, export it to S3, and download locally. You’ll need to set a couple of BASH variables and edit a file (which could really be automated as well: [https://stackoverflow.com/questions/11145270/how-to-replace-an-entire-line-in-a-text-file-by-line-number](https://stackoverflow.com/questions/11145270/how-to-replace-an-entire-line-in-a-text-file-by-line-number)).

```
export REDIS_NAME="redis-staging"
export SNAPSHOT_NAME="redis-staging-001
BUCKET='s3://elasticache-backups.dev.io/'
LOCAL_DIR='~/redis/'
```

Keep in mind here I’m storing all .rdb’s in `~/redis`, you may want to store your dumps and configs elsewhere (or maybe you already do!)

```
echo 'Initializing Snapshot.'
aws elasticache create-snapshot --cache-cluster-id $REDIS_NAME --snapshot-name $SNAPSHOT_NAME 2>&1 >/dev/null

# Loops to check when the snapshot is available.
echo 'Creating Snapshot'
CREATING=$(aws elasticache describe-snapshots --snapshot-name $SNAPSHOT_NAME --query "Snapshots[*].[SnapshotStatus][0][0]")

# Removing double quotes, otherwise our while loop won't work.
CREATING="${CREATING//\"}"

while [ "$CREATING" != "available" ]; do
  sleep 3
  printf '.' &
  CREATING=$(aws elasticache describe-snapshots --snapshot-name    $SNAPSHOT_NAME --query "Snapshots[*].[SnapshotStatus][0][0]")
  CREATING="${CREATING//\"}"
done
```

Above, we’re creating a snapshot, and getting the status, which is what the `--query` is accomplishing. We also need to remove double quotes from our status in order for our `while` loop to work properly. Lastly, we check every 3 seconds for updates.

```
echo '' #Gives us a new line here.
echo 'Exporting Snapshot to S3'

aws elasticache copy-snapshot --source-snapshot-name $SNAPSHOT_NAME --target-snapshot-name $SNAPSHOT_NAME --target-bucket elasticache-backups.dev.io 2>&1 >/dev/null

# Loops to check if the snapshot exists in S3 yet.
FILE_EXISTS=$(aws s3 ls $BUCKET$SNAPSHOT_NAME)

while [ ! "$FILE_EXISTS" ]; do
  sleep 3
  printf '.' &
  FILE_EXISTS=$(aws s3 ls $BUCKET$SNAPSHOT_NAME)
done
```

Once the snapshot is available, we copy it to `S3`and wait for that to finish.

```
S3_FILENAME=$(aws s3 ls $BUCKET$SNAPSHOT_NAME | sort | tail -n 1 | awk '{print $4}')

# Downloads latest dump from S3.
echo 'Copying Snapshot to Local...'

# Will only download if we attempt no SSL first.
aws s3 cp "$BUCKET$S3_FILENAME" "$LOCAL_DIR$S3_FILENAME" --no-verify-ssl
aws s3 cp "$BUCKET$S3_FILENAME" "$LOCAL_DIR$S3_FILENAME"
echo 'Snapshot Now Available.'
```

Above, listing our `S3` files gives us tabular data, so we use `awk` to get the latest file name (our snapshot, boo-yah!). Lastly, you may or may not need to attempt `--no-verify-ssl`, but I did 😢

The last step is to edit your `~/redis/redis.conf` file to point to the Elasticache dump you just downloaded:

```
# The filename where to dump the DB
dbfilename redis-staging-0001.rdb

# The working directory
dir ./
```

Launch your Redis server locally now with: `redis-server ~/redis/redis.conf`.
