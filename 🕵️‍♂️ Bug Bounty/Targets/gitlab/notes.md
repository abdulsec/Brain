
here the command to to run it 

```
docker run --detach \
  --hostname gitlab.example.com \
  --publish 443:443 --publish 8086:80 --publish 2222:22 \
  --name gitlab-ee \
  --restart always \
  --volume ~/gitlab/config:/etc/gitlab \
  --volume ~/gitlab/logs:/var/log/gitlab \
  --volume ~/gitlab/data:/var/opt/gitlab \
  gitlab/gitlab-ee:latest

```

here the command to remove it 

```
docker rm gitlab-ee

```
