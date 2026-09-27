import mimetypes
import pathlib
import urllib.parse
import socket
from http.server import HTTPServer, BaseHTTPRequestHandler
from multiprocessing import Process
from datetime import datetime
from pymongo import MongoClient

HTTP_PORT = 3000
SOCKET_PORT = 5000
SOCKET_HOST = '127.0.0.1'

class HttpHandler(BaseHTTPRequestHandler):
    def do_GET(self):
        pr_url = urllib.parse.urlparse(self.path)
        if pr_url.path == '/':
            self.send_html_file('front-init/index.html')
        elif pr_url.path == '/message.html':
            self.send_html_file('front-init/message.html')
        else:
            file_path = pathlib.Path('front-init') / pr_url.path[1:]
            if file_path.exists():
                self.send_static(file_path)
            else:
                self.send_html_file('front-init/error.html', 404)

    def do_POST(self):
        data = self.rfile.read(int(self.headers['Content-Length']))
        
        client_socket = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
        client_socket.sendto(data, (SOCKET_HOST, SOCKET_PORT))
        client_socket.close()
        
        self.send_response(302)
        self.send_header('Location', '/')
        self.end_headers()

    def send_html_file(self, filename, status=200):
        self.send_response(status)
        self.send_header('Content-type', 'text/html')
        self.end_headers()
        with open(filename, 'rb') as fd:
            self.wfile.write(fd.read())

    def send_static(self, file_path):
        self.send_response(200)
        mt = mimetypes.guess_type(file_path)[0] or 'text/plain'
        self.send_header('Content-type', mt)
        self.end_headers()
        with open(file_path, 'rb') as fd:
            self.wfile.write(fd.read())

def run_http_server():
    server_address = ('0.0.0.0', HTTP_PORT)
    http = HTTPServer(server_address, HttpHandler)
    try:
        print(f"HTTP Server is running on port {HTTP_PORT}")
        http.serve_forever()
    except KeyboardInterrupt:
        http.server_close()

def run_socket_server():
    server_socket = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
    server_socket.bind(('0.0.0.0', SOCKET_PORT))
    
    client = MongoClient("mongodb://mongo:27017/")
    db = client.messages_db
    collection = db.messages

    try:
        print(f"Socket Server is running on port {SOCKET_PORT}")
        while True:
            data, address = server_socket.recvfrom(1024)
            parsed_data = dict(urllib.parse.parse_qsl(data.decode()))
            
            document = {
                "date": str(datetime.now()),
                "username": parsed_data.get('username'),
                "message": parsed_data.get('message')
            }
            collection.insert_one(document)
    except KeyboardInterrupt:
        server_socket.close()

if __name__ == '__main__':
    http_process = Process(target=run_http_server)
    socket_process = Process(target=run_socket_server)

    http_process.start()
    socket_process.start()

    http_process.join()
    socket_process.join()
