COMMON
------

find all files in /path/to/folder (and its subfolders) that contain the word "help" anywhere in the file name<br />
```
find /path/to/folder -name '*help*'
```

find all files in /path/to/files (and its subfolders) that contain the word "help" in the contents and display the path and file name of each match on a separate line<br />
```
grep -rnl '/path/to/files' -e 'help'
```

log in to remote ssh<br />
```
ssh -i /path/to/some.key user@host
```

add ssh key to allow logging in to remote ssh without supplying a path to the key<br />
```
ssh-add /path/to/some.key
```

mount remote ssh as a folder in the file browser after ssh key has been added<br />
```
ssh://user@host
```

launch an android emulator<br />
```
rsync -rv --exclude=ignore /sourc /destinationd ~/Android/Sdk/emulator
sudo ./emulator -avd Pixel_3a_API_30_x86 -read-only
```
recursively copy all files and folders from /source to /destination except folders names ignore<br />
```
rsync -rv --exclude=ignore /sourc /destination
```

FIND
----

perform the same find but show the line numbers of the matches (there may be multiple matches for the same file if the file contains the word "help" on more than one line)<br />
```
grep -rHino '/path/to/files' -e 'help' | cut -d: -f1-2
```

this will show each matching file once with no line numbers, this is equivalent to to: grep -rnl '/path/to/files' -e 'help'<br />
```
grep -rHinol '/path/to/files' -e 'help' | cut -d: -f1-2
```

deep find all files in /path/to/files (and its subfolders) that contain the word "help" in the contents<br />
```
grep -rno -R -d recurse '/path/to/files' -e 'help'
```
or
```
grep -Hno -R -d recurse '/path/to/files' -e 'help'
```

 find any lines that start with the word "help" (ignoring any leading whitespace by using \s*) and remove them<br />
```
.*^\s*help.*\r?\n
```

TASKS
-----

dump the contents of database on host to /server/dump/database<br />
```
mongodump --uri=mongodb+srv://user:pass@host/database --out=/server/dump/
```

restore the contents of /server/dump/database to database on localhost<br />
```
mongorestore --drop --db database /server/dump/database/
```

restore the contents of /server/dump/database to database on host<br />
```
mongorestore --host host --drop -db database /server/dump/database/
```

SYSTEM
------

kill all processes that contain the word "myapp"<br />
```
sudo pkill -f myapp
```

INSTALL
-------

install NVM<br />
```
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.1/install.sh | bash
```

install NodeJS with NVM<br />
```
curl -fsSL https://deb.nodesource.com/setup_17.x | sudo -E bash -
sudo apt-get install -y nodejs
```

install Docker<br />
```
sudo apt install docker.io
```

DOCKER
------

create image named mongodb from mongo image in Docker hub<br />
```
docker run -it -d -p 27017:27017 --net queues --name mongodb mongo
```

create project/mongo from new mongodb image<br />
```
sudo docker commit mongodb project/mongo
```

create image from project/mongo<br />
```
sudo docker run -it -d -p 27017:27017 --name mongodb project/mongo
```

optionally save data to external volume<br />
```
sudo docker run -it -d -p 27017:27017 --name mongodb -v /server/project/mongodb:/data/db project/mongo
```

build local docker image<br />
```
sudo docker build -t project/project .
```

create new container named project with options from local image exposing port 27017<br />
```
sudo docker run --link=mongodb -p 27017:27017 --name project -d project/project
```

start/Stop the new container named project<br />
```
sudo docker container start|stop project
sudo docker container start|stop mongodb
```

commit changes from container named project (container must be stopped) to project/project image:<br />
```
sudo docker commit project project/project
```

commit changes from container named project (container must be stopped) to new project/projectv2 image:<br />
```
sudo docker commit project project/projectv2
```

use docker compose to orchestrate the services<br />
```
sudo docker compose build|up|down
```

TOOLS
-----

open a remote desktop session to host as user on port at 1600x900 resolution<br />
```
rdesktop host:port -g 1600x900 -u user
```

open a remote desktop session to host as user on port at 1600x900 resolution and automatically login using password (nohup and the & allow the window to be closed but keep the session open)<br />
```
nohup rdesktop host:port -g 1600x900 -u user -p password &
```
