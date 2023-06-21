
## Step1: Installing Colimo
- Run install command
```
brew install colima
```
- Verify installation by running command
```
colima
```
  
### Step2: Installing Docker's cli
- Run command to install
```
brew install docker docker-compose
```
- Create a directory
```
mkdir -p ~/.docker/cli-plugins
```
- And, create following symlink
```
ln -sfn $(brew --prefix)/opt/docker-compose/bin/docker-compose ~/.docker/cli-plugins/docker-compose
```

- Test installation by running command
```
docker compose
```
