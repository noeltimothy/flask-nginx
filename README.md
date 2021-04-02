# Production ready Flask

## Simple Flask App

```
#app.py
from flask import Flask
app = Flask(__name__)

@app.route('/')
def default():
    return F"<html> Hello </html>"
    
if __name__ == "__main__":
    app.run(debug=False)
```    

## Architecture

### Option 1

*Pure Flask running forever*

```
sudo nohup python app.py > /var/log/app.log 2>&1 &
```

### Option 2

*Flask with Gunicorn*

You can expose your Flask app to the outside world using a WSGI server such as Gunicorn

- Flask runs on localhost and port 5000 by default
- Gunicorn is a WSGI server that runs on your public IP and port 80

*Install and setup Gunicorn*

```
sudo pip3 install gunicorn
```

Setup a simple Gunicorn service file called 'gunicorn.service'

```
[Unit]
Description=gunicorn daemon for /home/ubuntu/<your-flask-app-dir>
After=network.target

[Service]
User=ubuntu
Group=www-data
RuntimeDirectory=gunicorn
WorkingDirectory=/home/ubuntu/<your-flask-app-dir>
ExecStart=/usr/bin/sudo /usr/local/bin/gunicorn --bind 0.0.0.0:80 --workers=4 app:app --log-file gunicorn.log --capture-output --log-level debug --reload
ExecReload=/bin/kill -s HUP $MAINPID
ExecStop=/bin/kill -s TERM $MAINPID

[Install]
WantedBy=multi-user.target
```

Copy the Service file to /etc/systemd/system and start it

```
sudo cp gunicorn.service /etc/systemd/system/
sudo service gunicorn restart
```

*Pain points*

- Make sure your flask app.run() only happens under main in your app.py
- Remember to not have debug=True or else warnings of not being production ready will appear in gunicorn.log
- Use gunicorn.log as your logging mechanism via "logging.getLogger('gunicorn.error')" in your app.py and also specifying the log file in gunicorn.service file as shown above.

```
if __name__ == '__main__':
    gunicorn_logger = logging.getLogger('gunicorn.error')
    app.logger.handlers = gunicorn_logger.handlers
    app.logger.setLevel(gunicorn_logger.level)
    app.run()
```

- Make sure your gunicorn.service file runs Gunicorn using sudo since we need access to port 80

### Option 3
