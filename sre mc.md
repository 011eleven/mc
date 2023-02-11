```
#!/bin/bash

# 启动服务器的函数
start() {
    echo "正在启动 Minecraft 服务器..."
    java -Xmx1024M -Xms1024M -jar server.jar nogui
}

# 停止服务器的函数
stop() {
    echo "正在关闭 Minecraft 服务器..."
    screen -r mc -X stuff "stop^M"
}

# 重启服务器的函数
restart() {
    echo "正在重启 Minecraft 服务器..."
    stop
    sleep 10
    start
}

case "$1" in
    start)
        start
    ;;
    stop)
        stop
    ;;
    restart)
        restart
    ;;
    *)
        echo "用法: $0 {start|stop|restart}"
esac
```

保存为一个文件，例如 "mc-control.sh"。运行它：

```
$ chmod +x mc-control.sh   # 给脚本添加执行权限
$ ./mc-control.sh start    # 启动服务器
$ ./mc-control.sh stop     # 停止服务器
$ ./mc-control.sh restart  # 重启服务器

```

脚本中的 stop 函数中使用了 `screen -r mc -X stuff "stop^M"` 这句话来停止服务器，这个需要你在启动服务器之前，用 screen 来启动服务器。比如说你要用命令 'screen -S mc java -Xmx1024M -Xms1024M -jar server.jar nogui'来启动服务器

```
#!/bin/bash

# 读取用户名
echo "Please enter the username of the player you want to add:"
read username

# 获取 whitelist.json 的文件路径
whitelist_file="./whitelist.json"

# 如果文件不存在，创建文件
if [ ! -f "$whitelist_file" ]; then
    echo "{\"players\": []}" > "$whitelist_file"
fi

# 读取 JSON 文件
json=$(<"$whitelist_file")

# 解析 JSON 文件
players=$(echo $json | jq -r '.players')
operators=$(echo $json | jq -r '.operators')

# 判断用户名是否已经存在
if [[ $players == *"$username"* ]]; then
  echo "Error: $username is already in the whitelist"
  exit 1
fi

# 添加用户名
players="$players,\"$username\""

# 重新生成 JSON 文件
json=$(echo "{\"players\": $players,\"operators\": $operators}")

# 保存 JSON 文件
echo $json > "$whitelist_file"

echo "$username has been added to the whitelist"
```

这个脚本假设你已经安装了 jq 这个工具，它是一个命令行 JSON 处理器。如果你没有安装它，可以使用如下命令安装：

sudo apt-get install jq (on Ubuntu)

或者

yum install jq (on Centos)

备份脚本可能长这样：

```
#!/bin/bash

# 获取当前时间
now=$(date +"%Y-%m-%d-%H-%M")

# 压缩存档文件夹
tar -czvf /path/to/backup/worlds_$now.tar.gz /path/to/worlds
tar -czvf /path/to/backup/mods_$now.tar.gz /path/to/mods

```

编写另一个脚本实现回档功能:

```
#!/bin/bash

echo "Please enter the backup date (YYYY-MM-DD-HH-MM):"
read backup_date

echo "Restore worlds from backup: worlds_$backup_date.tar.gz"
tar -xzvf /path/to/backup/worlds_$backup_date.tar.gz -C /path/to/worlds
echo "Restore worlds finished"

echo "Rest
```

使用 `cron` 来实现定时自动备份的步骤如下:

1. 首先，需要编写一个备份脚本来备份你的存档文件和模组文件。如果你还没有编写备份脚本,请按照上面我提供的示例进行编写。
2. 使用命令 `crontab -e` 打开 `cron` 的配置文件。如果这是你第一次使用 `cron`，那么会新建一个配置文件。
3. 在配置文件中添加一行代码来配置定时备份任务。格式是分，时，日，月，星期，运行的命令。例如，如果你想每小时运行一次备份脚本,可以添加一行

```
0 * * * * /path/to/backup.sh
```

如果你想每天晚上10点进行备份，可以添加一行

```
0 22 * * * /path/to/backup.sh
```

下面是一个示例脚本，用于删除超过48小时的备份中的存档，并保留凌晨4点之后的备份：

```
#!/bin/bash

# 设置过期时间为48小时前
expired_time=$(date -d '-48 hours' +%s)
keep_time="04:00:00"

# 找到备份目录下所有文件
for file in /path/to/backup/*; do
  if [[ -f $file ]]; then
    # 判断文件是否过期
    file_time=$(date -r $file +%s)
    if [[ $file_time -lt $expired_time ]]; then
      file_date=$(date -r $file +%F)
      file_time=$(date -r $file +%T)
      if [[ $file_time > $keep_time ]]; then
        # 删除过期文件
        rm $file
        echo "Deleted $file"
      fi
    fi
  fi
done
```

你需要把上面的 /path/to/backup/* 改成你本地的文件路径

使用方法：

- 将代码保存为一个文本文件，例如 deletescript.sh

- 赋予运行权限 chmod +x deletescript.sh

- 编辑crontab -e 添加定时任务，如 0 */48 * * * /path/to/deletescript.sh

  

  

  

  超过48小时的备份中只挑选凌晨4点的保留下来

```
#!/bin/bash

#设置过期时间为48小时前
expired_time=$(date -d '-48 hours' +%s)
keep_time="04:00:00"

#先停止服务器
/path/to/server/script stop

#找到备份目录下所有文件
for file in /path/to/backup/*; do
    if [[ -f $file ]]; then
        #判断文件是否过期
        file_time=$(date -r $file +%s)
        if [[ $file_time -lt $expired_time ]]; then
          file_date=$(date -r $file +%F)
          file_time=$(date -r $file +%T)
          if [[ $file_time > $keep_time ]]; then
            #删除过期文件
            rm $file
            echo "Deleted $file"
          fi
        fi
    fi
done

#重启服务器
/path/to/server/script start
```

可以使用脚本来检测mod或插件的上传，并生成新版本。 下面是一个示例脚本，该脚本检测/path/to/mods目录下是否有新的mod或插件，如果有，就将它们拷贝到一个新的备份目录中，并生成一个新的版本号，最后只保留最新的版本。

```
#!/bin/bash

# 设置mod目录
mod_dir=/path/to/mods
# 设置备份目录
backup_dir=/path/to/backup
#设置服务器文件名
server_name=minecraft_server
#关闭服务器命令
stop_server="screen -S minecraft -X quit"
#启动服务器命令
start_server="screen -dmS minecraft java -jar ${server_name}.jar nogui"

# 关闭服务器
eval $stop_server
echo "服务器已经关闭"

# 获取当前版本号
if [[ -f $backup_dir/version ]]; then
    current_version=$(cat $backup_dir/version)
else
    current_version=0
fi

# 检查是否有新的mod
new_mod=0
for file in $mod_dir/*; do
    if [[ -f $file ]]; then
        # 检查文件是否已备份
        if [[ ! -f $backup_dir/$(basename $file) ]]; then
            new_mod=1
            break
        fi
    fi
done

# 如果有新的mod则生成新的版本号
if [[ $new_mod -eq 1 ]]; then
    current_version=$((current_version+1))
    echo $current_version > $backup_dir/version
    # 备份新的mod
    cp -r $mod_dir $backup_dir/version_$current_version
    echo "新版本 $current_version 已备份"
else
    echo "没有新的mod需要备份"
fi

# 重启服务器
eval $start_server
echo "服务器已经重启"
```

这个脚本可以通过以下步骤来使用：

1. 使用文本编辑器打开该脚本并更改 mod_dir 和 backup_dir 变量的值为你本地电脑上对应目录的路径。
2. 将该脚本拷贝到你本地电脑上并保存为 .sh文件
3. 打开终端并切换到该脚本的所在目录
4. 执行脚本，可以用下面的命令执行

```
Copy code
sh script_name.sh
```

 5.可以使用cronjob 来定时执行这个脚本，这样就不用手动执行了。







通过以下步骤将本地服务器连接到 VPS，让其他人通过 VPS 的 IP 访问服务器:

1. 在本地计算机上运行 Minecraft 服务器。
2. 在 VPS 上安装并运行 ssh 客户端。
3. 使用 ssh 客户端连接到 VPS。
4. 使用 ssh 客户端在 VPS 上运行 ssh -R 25565:localhost:25565 root@your-server-ip 这样可以将你本地的25565端口映射到vps上
5. 在 VPS 上启动 Minecraft 服务器。
6. 将 VPS 的 IP 地址告诉玩家，他们就可以通过该地址连接到服务器了。







在 Minecraft 服务器上设置白名单可以限制只有特定玩家才能连接到服务器。下面是设置白名单的步骤:

1. 在服务器文件夹中找到 server.properties 文件。
2. 在 server.properties 文件中找到 “white-list=false”，并将其改为 “white-list=true”。这样可以开启白名单功能。
3. 在服务器控制台输入“whitelist add <username>”来添加白名单玩家。如：whitelist add player1,whitelist add player2
4. 可以使用“whitelist list”命令查看白名单玩家列表
5. 可以使用“whitelist remove <username>”命令移除白名单玩家
6. 在服务器控制台输入 “save-all” 命令来保存更改。
7. 使用“reload”命令重新加载白名单，使更改生效。

注意：白名单功能需要在服务器端启动才能生效。如果白名单功能没有生效，请确保您已经在 server.properties 文件中将 white-list 设置为 true。





