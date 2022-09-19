# Definitions & General Information

Docker - Docker is a set of platform as a service products that use OS-level virtualization to deliver software in packages called containers. More information at:

https://www.docker.com/

IPFS - IPFS is The InterPlanetary File System is a protocol and peer-to-peer network for storing and sharing data in a distributed file system. More information at:

https://ipfs.io/

Seed - Seed is an identifier of the Stellar wallet. You must create it before you are starting to deal with a dockers

Nickname- Nickname is some user friendly name that is used in Tor. 

# Prerequisites

## Install docker:

    curl -fsSL https://get.docker.com -o get-docker.sh
    sh get-docker.sh
    chmod +x /usr/bin/docker
    sudo groupadd docker
    sudo usermod -aG docker $USER && newgrp docker

## Docker minimal requirements:

    Memory: 512MB RAM (2GB Recommended). Disk: Sufficient amount to run the Docker containers you wish to use. CPU: 2 cores

## Create workspace directory:

    torplusworkspace=<yourworkspacedir>
    mkdir -p ${torplusworkspace}
    cd ${torplusworkspace}

# Run IPFS client

## Pull image
    docker pull torplusdev/production:ipfs-latest

## Run Tor-Plus container with IPFS:

## create workspace

    cd ${torplusworkspace}
    nickname=tunick21 # set your nickname
    seed=SCR27IGKMKXSOKUV7AC4T3HBTBVBL2MI45HHFSDNRYJFFVKWQAWBBKKZ # set your seed
 
## run docker container
    docker run \
    --name torplusipfs \
    -p 28000:28080 \
    -e nickname=${nickname} \
    -e PP_ENV=prod \
    -e PP_SINGLEHOP_HS=1 \
    -e seed=${seed} \
    -v ${PWD}/tor:/root/tor \
    -v ${PWD}/ipfs:/root/.ipfs \
    -v ${PWD}/hidden_service:/root/hidden_service \
    --rm \
    torplusdev/production:ipfs-latest

## Add new files to ipfs

#Commands are executed in a new terminal with a running docker container on the computer

    torplusworkspace=<yourworkspacedir>
    cd ${torplusworkspace}/ipfs
    sudo mkdir -p ./data
    sudo cp ~/<the path to the file that we will upload to the ipfs network> /${torplusworkspace}/ipfs/data  # The file that we will upload to the ipfs is copied to the folder
    sudo docker exec -it torplusipfs /bin/bash
     ./ipfs add ~/.ipfs/data/<file name>  # After successfully uploading the file to the ipfs, a message is displayed "added <cid> <file name>"
     ./ipfs get "cid"

## Check uploaded files ipfs:

#Copy cid link from previous step. 

    Open Chrome browser with TorPlus installed on your computer and follow the link http://localhost:8080/ipfs/"cid"  
    # Playback or display of the file uploaded to the ipfs will start, if the file format is not supported by the site, 
    then it will be downloaded to the computer. Attention!!! This verification will be charged for using the TorPlus network!
    
## Setting and testing Video site:

    1. Get the <cid> of the uploaded video file to the ipfs network. # Downloadable video formats must be supported by the CMS site. For example for wordpress - mp4, m4v, webm, ogv, wmv, flv.
    2. Launch a docker container, as a result, get the website address in the TorPlus network - https://torplus.{your domain name}
    3. Go to the WordPress site as an administrator.
    4. Go to the site admin panel and add a new post or site page. # Or go to the video page.
![image](https://user-images.githubusercontent.com/52072466/144409412-cb7434d0-1523-4980-bb37-cc7641c4f5a1.png)

    5. When creating a page, click on the button "Toggle block inserter" and select a block "Video"
![image](https://user-images.githubusercontent.com/52072466/144410808-e10f8e80-db67-4bb2-95d9-23e06813e01d.png)
![image](https://user-images.githubusercontent.com/52072466/144410921-6ead500a-9822-4952-aff4-904ef575db13.png)

    6. When inserting a video block, select "Insert from URL", enter a link to the video file in the format "/ipfs/<cid>" click on the button "Upload" and click on "Publish" or "Update".
![image](https://user-images.githubusercontent.com/52072466/144411109-0769a596-7a6c-4b60-9894-cc768ed471c6.png)

    7. When you click on the "Preview" button, a WordPress site page and the added video will be displayed on the page.

# Run web site as host

## Create folder for ssl and copy ssl to dir

    torplusworkspace=<yourworkspacedir>
    mkdir -p ${torplusworkspace}
    cd ${torplusworkspace}
    mkdir -p ssl

## If use let's encrypt:

    #install certbot:
    sudo apt update && sudo apt install -y certbot

    domain=<yourdomains>
    email=<youremail>
    certbot certonly --standalone -d ${domain} \
                --non-interactive --agree-tos --email ${email} \
                --http-01-port=80

    cat /etc/letsencrypt/live/${domain}/fullchain.pem /etc/letsencrypt/live/${domain}/privkey.pem > ${torplusworkspace}/ssl/${domain}.pem

## Pull docker image:

    docker pull torplusdev/production:ipfs_haproxy-latest

## For host static files
    
    #Get the <cid> of the uploaded video file to the ipfs network. # Downloadable video formats must be supported by the CMS site. For example for wordpress - mp4, m4v, webm, ogv, wmv, flv.
    #Set static files:
        cd ${torplusworkspace}
        mkdir static 
        echo "<html>
                 <head>
                  <meta charset="utf-8">
                  <title>video</title>
                 </head>
                 <body>
                  <video width="400" height="300" controls="controls" controlsList="nodownload" autoplay muted>
                   <source src="/ipfs/insert your cid here">
                  </video>
                   <video width="400" height="300" controls="controls" controlsList="nodownload" autoplay muted>
                   <source src="/ipfs/insert your cid here">
                  </video>
                 </body>
               </html>" > ./static/index.html # or copy your static files
    #After starting the docker container, go to the page https://torplus.{your domain name} from the TorPlus client. 2 added video files are displayed
    
You can see the example [here](https://torplus.videotpdemo.com/wp-content/uploads/2022/07/helloworld.html) or download the finished html [here](https://github.com/torplusdev/helloworld.html)

## Run docker image:

    cd ${torplusworkspace}
    seed=SCR27IGKMKXSOKUV7AC4T3HBTBVBL2MI45HHFSDNRYJFFVKWQAWBBKKZ  # set your seed
    nickname=tum332 # set your nickname
    
## Run docker
    docker run \
        --name torplus \
        -e nickname=${nickname} \
        -e seed=${seed} \
        -e role=hs_client \
        -e HOST_PORT=80 \
        -e PP_ENV=prod \
        -e http_address=127.0.0.1:80 \
        -e useNginx=1 \
        -p 80:80 \
        -p 28000:28080 \
        -v ${PWD}/tor:/root/tor \
        -v ${PWD}/ipfs:/root/.ipfs \
        -v ${PWD}/ssl:/etc/ssl/torplus/ \
        -v ${PWD}/hidden_service:/root/hidden_service \
        -v ${PWD}/static:/var/www/html \
        --rm \
        torplusdev/production:ipfs_haproxy-latest

## Add text record to DNS:

    1. Echo onion address to console: 
    Run command:
    echo torplus=$(cat hidden_service/hsv3/hostname)
![image](https://user-images.githubusercontent.com/52072466/144443915-afac936e-7eb7-4304-a834-3c017e3d3633.png)
    
    2. Add it to DNS TXT records:
    Godaddy service (https://godaddy.com/):
![image](https://user-images.githubusercontent.com/52072466/147237710-78fab3ee-3698-4090-8b42-2ce4e32fb485.png)

    #Azure portal service:
    1. Find DNS zone in resources:
![image](https://user-images.githubusercontent.com/52072466/144444462-f22bd46f-b3a4-485f-8b40-95eaccc276b9.png)

    2. Add record set:
![image](https://user-images.githubusercontent.com/52072466/144444576-29546bca-449a-49fa-97c3-abb27d45a924.png)

    3. Check txt record. Open https://www.whatsmydns.net/#TXT/ Enter your domain name.
![image](https://user-images.githubusercontent.com/52072466/144444770-2c565917-52d3-4f49-ba28-ef4efafa0d0d.png)

## Add torplus subdomain to your domain

    In the domain manager you need to add forward subdomain of torplus to https://torplus.com

# Host from another ip or host or localhost site

## Create folder for ssl and copy ssl to dir

    torplusworkspace=<yourworkspacedir>
    mkdir -p ${torplusworkspace}
    cd ${torplusworkspace}
    mkdir -p ssl

## If use let's encrypt:

## install certbot:
    apt update && apt install -y certbot

    domain=<yourdomains>
    email=<youremail>
    certbot certonly --standalone -d ${domain} \
                --non-interactive --agree-tos --email ${email} \
                --http-01-port=80
    
    cat /etc/letsencrypt/live/${domain}/fullchain.pem /etc/letsencrypt/live/${domain}/privkey.pem > ${torplusworkspace}/ssl/${domain}.pem

## Pull docker image:

    docker pull torplusdev/production:ipfs_haproxy-latest

## Run docker container:

    cd ${torplusworkspace}
    seed=SCR27IGKMKXSOKUV7AC4T3HBTBVBL2MI45HHFSDNRYJFFVKWQAWBBKKZ # set your seed
    nickname=tum33212 # set your nickname
    http_address=1.1.1.1:80 # set your webserver ip/name. The port :80 must be entered after ip or domain

##  Run docker:
    docker run \
            --name torplus \
            -e nickname=${nickname} \
            -e seed=${seed} \
            -e role=hs_client \
            -e HOST_PORT=80 \
            -e PP_ENV=prod \
            -e http_address=${http_address} \
            -p 80:80 \
            -p 28000:28080 \
            -v ${PWD}/tor:/root/tor \
            -v ${PWD}/ipfs:/root/.ipfs \
            -v ${PWD}/ssl:/etc/ssl/torplus/ \
            -v ${PWD}/hidden_service:/root/hidden_service \
            --add-host host.docker.internal:host-gateway \
            --rm \
            torplusdev/production:ipfs_haproxy-latest

## Add text record to DNS:

    cat ${torplusworkspace}/hidden_service/hsv3/hostname
 
    torplus=<onion address without .onion suffix>

#To check TXT record you can use 
https://www.whatsmydns.net/#TXT/torplus.{domain}

## Add torplus subdomain to your domain

    In the domain manager you need to add forward subdomain of torplus to https://torplus.com
    
# Сhecking sample server

## Install TorPluse client

    1. Go to the site https://torplus.com/download/
    2. Select the installer for the OS (example "Download for Windows (10+)")
    3. Run installer.
    4. Follow the instructions of the installer.
    5. Start or restart (if open) Сhrome browser.
    6. In the browser message "Torplus extension add" click on the button Enable extension.
![image](https://user-images.githubusercontent.com/52072466/144256915-900f09a6-fb4f-4e52-ae57-55a3b9ce6900.png)
    
    7. Go to https://torplus.{your domain name}  # There is a successful transition to the site https://torplus.{your domain name}
    8. Сlick on the Torplus extension with the left mouse button and switch the switch position to off mode.
![image](https://user-images.githubusercontent.com/52072466/144257475-42afcdba-4e73-4ce2-aa27-adb83cfeedea.png)
    
    9. Go to https://torplus.{your domain name}  # There is a redirect to the page for requires the Torplus application https://torplus.com/requires/


