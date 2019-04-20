# Live Stream

A real, working live stream over RMTP protocol using Apache,node.js and Nginx 

# Configuration and usage

  - Download and install nginx
  - Go to the path where you have installed nginx, Go to conf directory, open ```nginx.conf``` or create it if not exists
  - install an Apache server and run it
  - download and put the content of **public_html** into your Apache server index directory
  - clear the file content and paste the following configuration into the file and save it
     ```
    worker_processes  1;
    
    
    events {
        worker_connections  1024;
    }
    
    
    rtmp {
        server {
            listen 1935;
    
            application live {
                live on;
    
                deny play all;
    
                push rtmp://127.0.0.1:1935/hls;
    
                on_publish http://127.0.0.1:8000/;
                on_publish_done http://127.0.0.1:8000/;
            }
    
            application hls {
                live on;
    
                allow publish 127.0.0.1;
                deny publish all;
                deny play all;
    
                hls on;
                hls_path path_to_public_html/live;
                hls_nested on;
            }
        }
    }
    ```
    **_Notice:_** don't forget to change __replace ```path_to_public_html``` with absolute path to where you put the content of **public_html**
  - download the node.js app, install it and run server.js
  - download and install OBS (download from here)
  - open obs, go to settings->stream
  - set RTMP url on __rtmp://your.host/live__
  - set the stream key on whatever you want and apply settings
  - go back and hit **Start Streaming**
  - after you started streaming, open your browser and type __http://your.host__, you'll see a list of live streams on your server, click on one of them to play
  - BOM ! you have implemented a live steram
# How It Works?
- OBS records the stream and sends it to RTMP host that you enter in settings (on port 1935 which Nginx is listening)
- Nginx recieves the stream, triggers publish event, which is sending stream data (not content of stream, jsut general data like stream name, the event name and ...) to _http://127.0.0.1:8000 which node.js server is listening on
    ```on_publish http://127.0.0.1:8000/;```
- node.js server gets the data, checks event and sends response to Nginx, Nginx catches data from __HTTP headers__, so that if you send 3** status with Location header, Nginx will publish the incoming stream on the value of Location header
consider that when you set on_publish event in Nginx server config, Nginx will wait for response from __http://127.0.0.1:8000/__ after getting each stream, if it recieves sttus code 3** will redirect stream to the name passed in __Location header__, if 2** it will continue RTMP session and otherwise, RTMP connection will drop
also note that server.js defines an array named ```streams``` which holds name of live streams, the array is pushed on each ```on_publish``` and is spliced on each ```on_publish_done``` (triggers when stream ends) event
- after node.js server responds, Nginx will behave based on headers as described, since we have the line :
```hls_nested on;```
    in config file, Nginx will create HLS manifest and segments in the path specified by this line :
```hls_path path_to_public_html/live;```
    and in a directory with the name server.js returns in __Location header__
- now, when you enter __http://your.host__ in your browser, apache returns index.html which then loads index.js, then javascript sends XHR HTTP to port 333 which server.js is listening and will return a JSON encoded array of ```streams```
- JS decodes string and displays live streams in links like :
    ```http://your.host/display.html?name=<stream_name>```
- by clicking on any link, you'll go to ```display.html``` page which has a ```display.js``` aside
- finally ```display.js``` gets stream name and sets 
```http://your.host/live/<stream_name>/index.m3u8``` 
as HLS manifest and finally HLS gets segments and play them by turn
# Notice
This app is implemented just to give you initial image of how you can create a live streaming website with no security in mind, before you use this in your products, you should add security aspects to it
