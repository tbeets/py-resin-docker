# Hello World test of resin.io, a Raspberry Pi 2, and a Python/Flask web app for the device

# Pre-Requisites

* GitHub.com account and git on your dev machine (Mac in my case...)
* GitHub.com registered public SSH key
* [Supported target device](http://docs.resin.io/#/pages/hardware/devices.md), in my case a Raspberry Pi 2
* Empty Micro-SD card (for the resin.io device image)
* Sign up for [resin.io](http://resin.io)

# Device bootstrap

* Create a logical application and declare target device, in my case `helloworld1` and Raspberry Pi 2
* Note that resin.io provides a git repo target that you use later to push your app
* Download the device image (in Raspberry Pi 2 case this is about 1.5 GB download)
* Note that you select wired or wi-fi (enabled) device image. For wi-fi there are options for SSID and network password (exact details specific to the device).
* Write the image to your empty Micro-SD (In my case I grabbed [Pi-Filler](http://ivanx.com/raspberrypi/) for this, but can use Mac/Linux commands...)
* Insert card in your device, and power up.
* After a few minutes, your device should be connected to resin.io and show in the resin.io web console for your application.

# Deploy a Python/Flask app container

## Write your device app (in my case I just pull the [Python/Flask sample](https://github.com/resin-io-projects/simple-server-python))

    /Users/tbeets/projects/learnResinio [tbeets@tbeets-mac] [16:50]
    > git clone git@github.com:resin-projects/simple-server-python.git
    Cloning into 'simple-server-python'...
    Warning: Permanently added the RSA host key for IP address '192.30.252.128' to the list of known hosts.
    remote: Counting objects: 52, done.
    remote: Total 52 (delta 0), reused 0 (delta 0), pack-reused 52
    Receiving objects: 100% (52/52), 704.78 KiB | 1.03 MiB/s, done.
    Resolving deltas: 100% (20/20), done.
    Checking connectivity... done.
    
## Note typical Python code and library files, plus a Dockerfile!

    /Users/tbeets/projects/learnResinio/simple-server-python [git::master] [tbeets@tbeets-mac] [18:31]
    > ls
    Dockerfile.template  README.md            img/                 requirements.txt     src/
    
## Dockerfile is familiar goodness (build steps for container image) accept for a special FROM clause

    /Users/tbeets/projects/learnResinio/simple-server-python [git::master] [tbeets@tbeets-mac] [19:08]
    > cat Dockerfile.template
    FROM resin/%%RESIN_MACHINE_NAME%%-python
    
    MAINTAINER Shaun Mulligan <shaun@resin.io>
    
    #switch on systemd init system in container
    ENV INITSYSTEM on
    
    # pip install python deps from requirements.txt
    # For caching until requirements.txt changes
    COPY ./requirements.txt /requirements.txt
    RUN pip install -r /requirements.txt
    
    COPY . /usr/src/app
    WORKDIR /usr/src/app
    
    #run main.py when the container starts
    CMD ["python","src/main.py"]

## Connect our git repo with the target resin.io app and connected devices from previous

    /Users/tbeets/projects/learnResinio/simple-server-python [git::master] [tbeets@tbeets-mac] [18:31]
    > git remote add resin todd@git.resin.io:todd/helloworld1.git

## Deploy to devices! (image is built on resin.io cloud but we can see the build log in real time)

    /Users/tbeets/projects/learnResinio/simple-server-python [git::master] [tbeets@tbeets-mac] [18:36]
    > git push resin master
    The authenticity of host 'git.resin.io (54.165.162.194)' can't be established.
    RSA key fingerprint is 14:bd:c0:8a:d5:c3:74:43:cd:2f:9e:f8:f5:e2:e2:a0.
    Are you sure you want to continue connecting (yes/no)? yes
    Warning: Permanently added 'git.resin.io,54.165.162.194' (RSA) to the list of known hosts.
    Counting objects: 52, done.
    Delta compression using up to 4 threads.
    Compressing objects: 100% (27/27), done.
    Writing objects: 100% (52/52), 704.78 KiB | 0 bytes/s, done.
    Total 52 (delta 20), reused 52 (delta 20)
    
    Starting build for todd/helloworld1, user todd
    Building on 'local'
    -----> Dockerfile template application detected
    Step 0 : FROM resin/raspberrypi2-python
     ---> ae5f214d5c50
    Step 1 : MAINTAINER Shaun Mulligan <shaun@resin.io>
     ---> Running in c5203cbcca50
     ---> ca3e3835a2ec
    Removing intermediate container c5203cbcca50
    Step 2 : ENV INITSYSTEM on
     ---> Running in 04f0a3218c2b
     ---> 2e23440147cf
    Removing intermediate container 04f0a3218c2b
    Step 3 : COPY ./requirements.txt /requirements.txt
     ---> 17fdb7e70fb1
    Removing intermediate container 2ac61755a2a3
    Step 4 : RUN pip install -r /requirements.txt
     ---> Running in 0a32bc2f2228
    Collecting Flask==0.10.1 (from -r /requirements.txt (line 1))
      Downloading Flask-0.10.1.tar.gz (544kB)
    Collecting Werkzeug>=0.7 (from Flask==0.10.1->-r /requirements.txt (line 1))
      Downloading Werkzeug-0.11.3-py2.py3-none-any.whl (305kB)
    Collecting Jinja2>=2.4 (from Flask==0.10.1->-r /requirements.txt (line 1))
      Downloading Jinja2-2.8-py2.py3-none-any.whl (263kB)
    Collecting itsdangerous>=0.21 (from Flask==0.10.1->-r /requirements.txt (line 1))
      Downloading itsdangerous-0.24.tar.gz (46kB)
    Collecting MarkupSafe (from Jinja2>=2.4->Flask==0.10.1->-r /requirements.txt (line 1))
      Downloading MarkupSafe-0.23.tar.gz
    Installing collected packages: Werkzeug, MarkupSafe, Jinja2, itsdangerous, Flask
      Running setup.py install for MarkupSafe
      Running setup.py install for itsdangerous
      Running setup.py install for Flask
    Successfully installed Flask-0.10.1 Jinja2-2.8 MarkupSafe-0.23 Werkzeug-0.11.3 itsdangerous-0.24
    You are using pip version 7.1.2, however version 8.0.2 is available.
    You should consider upgrading via the 'pip install --upgrade pip' command.
     ---> 309bfc983402
    Removing intermediate container 0a32bc2f2228
    Step 5 : COPY . /usr/src/app
     ---> 81046d6d976b
    Removing intermediate container c1cafcbcd1bb
    Step 6 : WORKDIR /usr/src/app
     ---> Running in 0b8640d17b57
     ---> 9996a097a8c0
    Removing intermediate container 0b8640d17b57
    Step 7 : CMD python src/main.py
     ---> Running in 40f57c722382
     ---> 70d5e92278f1
    Removing intermediate container 40f57c722382
    Successfully built 70d5e92278f1
    
    -----> Image created successfully!
    -----> Uploading image to registry...
    Image 38d98048753e already pushed, skipping - 0% total
    Image 6c38dfcb6ace already pushed, skipping - 0% total
    Image 4533c733949a already pushed, skipping - 14% total
    Image dc7d8102da8f already pushed, skipping - 14% total
    Image e4c80cc4e667 already pushed, skipping - 14% total
    Image d9e095f0fe59 already pushed, skipping - 14% total
    Image 6906b07fe485 already pushed, skipping - 14% total
    Image 18b9f417a970 already pushed, skipping - 14% total
    Image 342c1e58ff47 already pushed, skipping - 14% total
    Image 63e5ca9f7c4a already pushed, skipping - 14% total
    Image 1cd73a394d44 already pushed, skipping - 14% total
    Image 42338f999bb6 already pushed, skipping - 14% total
    Image e786133579ce already pushed, skipping - 14% total
    Image c0079ac125bd already pushed, skipping - 14% total
    Image d3b5b28936bd already pushed, skipping - 14% total
    Image 8be03e0c5f40 already pushed, skipping - 14% total
    Image e208e9c97e5d already pushed, skipping - 14% total
    Image e094cf9d9d33 already pushed, skipping - 21% total
    Image 9a7c6239ea6d already pushed, skipping - 35% total
    Image 294a8807f8b5 already pushed, skipping - 70% total
    Image 995242e87f81 already pushed, skipping - 79% total
    Image 4af867a0d4b0 already pushed, skipping - 79% total
    Image 46111a6df515 already pushed, skipping - 79% total
    Image a8d304998d4d already pushed, skipping - 79% total
    Image c71c9eb9d144 already pushed, skipping - 79% total
    Image 8ab1884c07af already pushed, skipping - 85% total
    Image 2a01928e7c01 already pushed, skipping - 85% total
    Image 0ebfd4c5a1f1 already pushed, skipping - 85% total
    Image 57c09d308162 already pushed, skipping - 85% total
    Image eb93d765627b already pushed, skipping - 87% total
    Image cfd9eb53c85c already pushed, skipping - 98% total
    Image 355456e7b249 already pushed, skipping - 98% total
    Image 814e52faecfe already pushed, skipping - 98% total
    Image d642607cc37c already pushed, skipping - 98% total
    Image 871dd9ed46c0 already pushed, skipping - 98% total
    Image 9c3900a93463 already pushed, skipping - 98% total
    Image ae5f214d5c50 already pushed, skipping - 98% total
    [==================================================>] 1.024 kB/1.024 kB - 98%
    [==================================================>] 1.024 kB/1.024 kB - 98%
    [==================================================>] 2.048 kB/2.048 kB - 98%
    [=================================================> ] 5.947 MB/5.994 MB - 99%
    [=================================================> ] 425.5 kB/429.6 kB - 100%
    [==================================================>] 1.024 kB/1.024 kB - 100%
    [==================================================>] 1.024 kB/1.024 kB - 100%
           Image uploaded successfully!
           Check your dashboard for device download progress: https://dashboard.resin.io/apps/42108/devices
    Build took 54 seconds
              \
               \
                \\
                 \\
                  >\/7
              _.-(6'  \
             (=___._/` \
                  )  \ |
                 /   / |
                /    > /
               j    < _\
           _.-' :      ``.
           \ r=._\        `.
          <`\\_  \         .`-.
           \ r-7  `-. ._  ' .  `\
            \`,      `-.`7  7)   )
             \/         \|  \'  / `-._
                        ||    .'
                         \\  (
                          >\  >
                      ,.-' >.'
                     <.'_.''
                       <'
    
    To todd@git.resin.io:todd/helloworld1.git
     * [new branch]      master -> master
    

## Test our deployed device container

After web console shows download complete, we test on our local LAN (note that `bernie-wired` is a local `/etc/hosts` alias for my wired Pi):

    /Users/tbeets/projects/learnResinio/simple-server-python [git::master] [tbeets@tbeets-mac] [18:54]
    > http http://bernie-wired
    HTTP/1.0 200 OK
    Content-Length: 12
    Content-Type: text/html; charset=utf-8
    Date: Thu, 04 Feb 2016 02:55:22 GMT
    Server: Werkzeug/0.11.3 Python/2.7.11
    
    Hello World!