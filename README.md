# n8n-server
n8n をセルフホストする

# Getting Started by Docker

> ref: https://docs.n8n.io/hosting/installation/docker/

## Docker Volume をコンテナ内の `/home/node/.n8n` に作成して起動

### 起動

```bash
docker volume create n8n_data
docker run -it --rm --name n8n -p 5678:5678 -v n8n_data:/home/node/.n8n docker.n8n.io/n8nio/n8n
```

> **NOTE**  
> docker volume create n8n_data を二度実行すると、最初の実行でボリューム n8n_data が作成され、二度目の実行では何も変更されずに終了します。既に存在するボリュームを再作成するのではなく、既存のボリューム名が返されます。

### バックアップを作成する

`$HOME/docker-volumes-backup/${VOLUME_NAME}_backup_$TIMESTAMP.tar` にバックアップを作成する:

```bash
#!/bin/bash

# バックアップを保存するディレクトリを指定
BACKUP_DIR="$HOME/docker-volumes-backup"
VOLUME_NAME="n8n_data"
TIMESTAMP=$(date +%Y%m%d_%H%M%S)
BACKUP_FILE="$BACKUP_DIR/${VOLUME_NAME}_backup_$TIMESTAMP.tar"

# バックアップディレクトリを作成
mkdir -p $BACKUP_DIR

# Dockerボリュームをtarアーカイブにバックアップ
docker run --rm -v $VOLUME_NAME:/volume -v $BACKUP_DIR:/backup busybox tar cvf /backup/$(basename $BACKUP_FILE) /volume

# バックアップ完了メッセージ
echo "Backup of volume '$VOLUME_NAME' completed. File saved as $BACKUP_FILE"
```

### バックアップを適用する

`$HOME/docker-volumes-backup/${VOLUME_NAME}_backup_YYYYMMDD_HHMMSS.tar` のバックアップを適用する:

```bash
#!/bin/bash

# バックアップファイルのディレクトリとファイル名を指定
BACKUP_DIR="$HOME/docker-volumes-backup"
VOLUME_NAME="n8n_data"
BACKUP_FILE="$BACKUP_DIR/${VOLUME_NAME}_backup_YYYYMMDD_HHMMSS.tar"  # 実際のバックアップファイル名に置き換えてください

# Dockerボリュームの作成（存在しない場合）
docker volume create $VOLUME_NAME

# バックアップファイルをDockerボリュームに復元
docker run --rm -v $VOLUME_NAME:/volume -v $BACKUP_DIR:/backup busybox tar xvf /backup/$(basename $BACKUP_FILE) -C /volume

# 復元完了メッセージ
echo "Restoration of volume '$VOLUME_NAME' from backup file $BACKUP_FILE completed."
```

## ローカルのディレクトリをマウントして起動

```bash
docker run -it --rm --name n8n -p 5678:5678 -v "$HOME/.n8n:/home/node/.n8n" docker.n8n.io/n8nio/n8n
```