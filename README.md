# memo
memo
----
#!/bin/bash

# 本物のdockerコマンドのパス（which docker で確認したもの）
REAL_DOCKER="/usr/bin/docker"

# "docker exec <コンテナ名> ..." の形式で呼ばれた場合のみロックをかける
if [ "$1" = "exec" ]; then
    # オプション（-it, -d, -u root など）を飛ばしてコンテナ名を特定する
    CONTAINER_NAME=""
    for arg in "${@:2}"; do
        case "$arg" in
            -*) continue ;;
            *) CONTAINER_NAME="$arg"; break ;;
        esac
    done

    # コンテナ名が取得できた場合、コンテナ名ごとのロックファイルで排他制御
    if [ -n "$CONTAINER_NAME" ]; then
        LOCK_FILE="/tmp/docker_exec_${CONTAINER_NAME}.lock"
        exec flock "$LOCK_FILE" "$REAL_DOCKER" "$@"
    fi
fi

# "docker exec" 以外（docker ps, docker run 等）はロックなしでそのまま実行
exec "$REAL_DOCKER" "$@"
