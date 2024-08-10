# n8n-server
n8n をセルフホストする

# Getting Started by Docker

> ref: https://docs.n8n.io/hosting/installation/docker/

### パターン1: Docker Volume をコンテナ内の `/home/node/.n8n` に作成してセットアップ

**起動**

```bash
docker volume create n8n_data
docker run -it --rm --name n8n -p 5678:5678 -v n8n_data:/home/node/.n8n -e N8N_SECURE_COOKIE=false docker.n8n.io/n8nio/n8n
```

> **NOTE**  
> docker volume create n8n_data を二度実行すると、最初の実行でボリューム n8n_data が作成され、二度目の実行では何も変更されずに終了します。既に存在するボリュームを再作成するのではなく、既存のボリューム名が返されます。

**バックアップを作成する**

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

**バックアップを適用する**

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

### パターン2: ローカルのディレクトリをマウントして起動

```bash
docker run -it --rm --name n8n -p 5678:5678 -v "$HOME/.n8n:/home/node/.n8n" -e N8N_SECURE_COOKIE=false docker.n8n.io/n8nio/n8n
```

# Note

**Docker コンテナを管理者権限なしで実行する**

```bash
sudo groupadd docker
sudo usermod -aG docker ${USER}
newgrp docker
```

**docker volume を消去**

```bash
docker volume rm n8n_data
```

**別マシンで作成したバックアップ用のtarファイルを別のマシンに転送して適用する**

送信元マシンでの作業

```bash
# カレントディレクトリにバックアップファイルを作成 (/path/.n8n/* にデータが保存されている場合)
tar -cvf ./backup_n8n.tar -C /path/.n8n .

# tarファイルを適用先マシンに送信する
scp backup.tar user@remote_host:/path/to/destination
# ex. scp /n8n_data_backup_20240101_000000.tar remote_username@x.x.x.x:/home/username
```

送信先マシンでの作業

- 先に送信先マシンで現在の Docker Volume をバックアップする。手順は「バックアップを作成する」を参照
- 「docker volume を消去」の手順でvolumeを削除する
- `docker volume create $VOLUME_NAME` でボリュームを作成する（ex. `docker volume create n8n_data` ）
- 「バックアップを適用する」の手順で送信元マシンで作成したバックアップファイルをvolumeに適用する
- dockerコンテナからファイルの閲覧権限がないケースがあるので以下で権限を与える

```bash
ID=$(id -u your_username)
GROUP=$(id -g your_username)
docker run --rm -v n8n_data:/volume -v $(pwd):/backup busybox chown -R $ID:$GROUP /volume
```
