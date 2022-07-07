# Rough comparison of install methods

```
# yarn install on virtual filesystem
docker run --rm -it -v "${PWD}/code:/code" -w "/code" node:14.19.3 yarn install
rm -rf code/node_modules

# yarn install on OverlayFS
docker run -d --rm --name yarn_installer -v "${PWD}/code:/code" -w "/code" node:14.19.3 sleep infinity
docker exec yarn_installer mkdir /target
docker exec -it yarn_installer yarn install --modules-folder "/target/node_modules"
docker stop yarn_installer

# yarn install on ext4
docker volume create install_target
docker run --rm -it -v "${PWD}/code:/code" -v "install_target:/target" -w "/code" node:14.19.3 yarn install --modules-folder "/target/node_modules"
docker volume rm install_target

# yarn install on ext4 with Mutagen sync running
docker volume create install_target
docker run -d --rm --name yarn_installer -v "${PWD}/code:/code" -v "install_target:/target" -w "/code" node:14.19.3 sleep infinity
mutagen sync create --name yarnsync ./target docker://yarn_installer/target
docker exec -it yarn_installer yarn install --modules-folder "/target/node_modules"
mutagen sync terminate yarnsync
docker stop yarn_installer
docker volume rm install_target
rm -rf ./target

# yarn install on ext4 with Mutagen sync running (via Mutagen Compose)
mutagen-compose run -i --rm web yarn install --modules-folder "/target/node_modules"
mutagen-compose down --rmi=local --volumes
rm -rf ./target
```
