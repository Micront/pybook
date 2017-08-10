# Лабораторная работа №11. Простой асинхронный WSGI-сервер

В этой лабораторной работе вашей задачей будет написать простой асинхронный wsgi-сервер.

```py
import socket

def main(host='localhost', port=9090):
    serversocket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    serversocket.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, True)
    serversocket.bind((host, port))
    serversocket.listen(5)

    while True:
        clientsocket, (client_address, client_port) = serversocket.accept()
        print(f"New client {client_address}:{client_port}")

        while True:
            try:
                data = clientsocket.recv(1024)
                print(f"Recv: {data}")
            except OSError:
                break

            if len(data) == 0:
                break

            sent_data = data
            while True:
                sent_len = clientsocket.send(data)
                if sent_len == len(data):
                    break
                sent_data = sent_data[sent_len:]
            print(f"Send: {data}")

        clientsocket.close()
        print(f"Bye-bye: {client_address}:{client_port}")


if __name__ == "__main__":
    main()
```

```py
import socket
import threading

def client_handler(sock, address, port):
    while True:
        try:
            data = sock.recv(1024)
            print(f"Recv: {data} from {address}:{port}")
        except OSError:
            break

        if len(data) == 0:
            break

        sent_data = data
        while True:
            sent_len = sock.send(data)
            if sent_len == len(data):
                break
            sent_data = sent_data[sent_len:]
        print(f"Send: {data} to {address}:{port}")

    sock.close()
    print(f"Bye-bye: {address}:{port}")

def main(host='localhost', port=9090):
    serversocket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    serversocket.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, True)
    serversocket.bind((host, port))
    serversocket.listen(5)

    while True:
        try:
            client_sock, (client_address, client_port) = serversocket.accept()
            print(f"New client {client_address}:{client_port}")
            client_thread = threading.Thread(target=client_handler,
                args=(client_sock, client_address, client_port))
            client_thread.daemon = True
            client_thread.start()

if __name__ == "__main__":
    main()
```

```py
import socket
import threading
import time

def worker_thread(serversocket):
    while True:
        clientsocket, (client_address, client_port) = serversocket.accept()
        print(f"New client {client_address}:{client_port}")

        while True:
            try:
                data = clientsocket.recv(1024)
                print(f"Recv: {data} from {client_address}:{client_port}")
            except OSError:
                break

            if len(data) == 0:
                break

            sent_data = data
            while True:
                sent_len = clientsocket.send(data)
                if sent_len == len(data):
                    break
                sent_data = sent_data[sent_len:]
            print(f"Send: {data} to {client_address}:{client_port}")

        clientsocket.close()
        print(f"Bye-bye: {client_address}:{client_port}")

def main(host='localhost', port=9090):
    serversocket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    serversocket.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, True)
    serversocket.bind((host, port))
    serversocket.listen(5)

    NUMBER_OF_THREADS = 2
    for _ in range(NUMBER_OF_THREADS):
        thread = threading.Thread(target=worker_thread,
            args=(serversocket,))
        thread.daemon = True
        thread.start()

    while True:
        time.sleep(1)

if __name__ == "__main__":
    main()
```

```py
import socket
import multiprocessing
import time

def worker_process(serversocket):
    while True:
        clientsocket, (client_address, client_port) = serversocket.accept()
        print(f"New client {client_address}:{client_port}")

        while True:
            try:
                data = clientsocket.recv(1024)
                print(f"Recv: {data} from {client_address}:{client_port}")
            except OSError:
                break

            if len(data) == 0:
                break

            sent_data = data
            while True:
                sent_len = clientsocket.send(data)
                if sent_len == len(data):
                    break
                sent_data = sent_data[sent_len:]
            print(f"Send: {data} to {client_address}:{client_port}")

        clientsocket.close()
        print(f"Bye-bye: {client_address}:{client_port}")

def main(host='localhost', port=9090):
    serversocket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    serversocket.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, True)
    serversocket.bind((host, port))
    serversocket.listen(5)

    NUMBER_OF_PROCESS = multiprocessing.cpu_count()
    print(f"Number of processes {NUMBER_OF_PROCESS}")
    for _ in range(NUMBER_OF_PROCESS):
        process = multiprocessing.Process(target=worker_process,
            args=(serversocket,))
        process.daemon = True
        process.start()

    while True:
        time.sleep(1)

if __name__ == "__main__":
    main()
```

```py
import socket
import threading
import multiprocessing
import time

def worker_thread(serversocket):
    while True:
        clientsocket, (client_address, client_port) = serversocket.accept()
        print(f"New client {client_address}:{client_port}")

        while True:
            try:
                data = clientsocket.recv(1024)
                print(f"Recv: {data} from {client_address}:{client_port}")
            except OSError:
                break

            if len(data) == 0:
                break

            sent_data = data
            while True:
                sent_len = clientsocket.send(data)
                if sent_len == len(data):
                    break
                sent_data = sent_data[sent_len:]
            print(f"Send: {data} to {client_address}:{client_port}")

        clientsocket.close()
        print(f"Bye-bye: {client_address}:{client_port}")

def worker_process(serversocket):
    NUMBER_OF_THREADS = 10
    for _ in range(NUMBER_OF_THREADS):
        thread = threading.Thread(target=worker_thread,
            args=(serversocket,))
        thread.daemon = True
        thread.start()

    while True:
        time.sleep(1)

def main(host='localhost', port=9090):
    serversocket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    serversocket.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, True)
    serversocket.bind((host, port))
    serversocket.listen(5)

    NUMBER_OF_PROCESS = multiprocessing.cpu_count()
    print(f"Number of processes {NUMBER_OF_PROCESS}")
    for _ in range(NUMBER_OF_PROCESS):
        process = multiprocessing.Process(target=worker_process,
            args=(serversocket,))
        process.daemon = True
        process.start()

    while True:
        time.sleep(1)

if __name__ == "__main__":
    main()
```

```py
import socket
import select

read_waiters = {}
write_waiters = {}
connections = {}

def accept_handler(serversocket):
    clientsocket, (client_address, client_port) = serversocket.accept()
    clientsocket.setblocking(False)
    print(f"New client: {client_address}:{client_port}")
    connections[clientsocket.fileno()] = (clientsocket, client_address, client_port)
    read_waiters[clientsocket.fileno()] = (recv_handler, (clientsocket.fileno(),))
    read_waiters[serversocket.fileno()] = (accept_handler, (serversocket,))

def recv_handler(fileno):
    def terminate():
        del connections[clientsocket.fileno()]
        clientsocket.close()
        print(f"Bye-Bye: {client_address}:{client_port}")

    clientsocket, client_address, client_port = connections[fileno]

    try:
        message = clientsocket.recv(1024)
    except OSError:
        terminate()
        return

    if len(message) == 0:
        terminate()
        return

    print(f"Recv: {message} from {client_address}:{client_port}")
    write_waiters[fileno] = (send_handler, (fileno, message))

def send_handler(fileno, message):
    clientsocket, client_address, client_port = connections[fileno]
    sent_len = clientsocket.send(message)
    print("Send: {} to {}:{}".format(message[:sent_len], client_address, client_port))
    if sent_len == len(message):
        read_waiters[clientsocket.fileno()] = (recv_handler, (clientsocket.fileno(),))
    else:
        write_waiters[fileno] = (send_handler, (fileno, message[sent_len:]))

def main(host='localhost', port=9090):
    serversocket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    serversocket.setblocking(False)
    serversocket.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, True)
    serversocket.bind((host, port))
    serversocket.listen(128)

    read_waiters[serversocket.fileno()] = (accept_handler, (serversocket,))
    while True:
        rlist, wlist, _ = select.select(read_waiters.keys(), write_waiters.keys(), [], 60)

        for r_fileno in rlist:
            handler, args = read_waiters.pop(r_fileno)
            handler(*args)

        for w_fileno in wlist:
            handler, args = write_waiters.pop(w_fileno)
            handler(*args)

if __name__ == "__main__":
    main()
```

### Асинхронный HTTP-сервер

```py
import asyncore
import asynchat
import socket
import multiprocessing
import logging
import mimetypes
import os
from urlparse import parse_qs
import urllib
import argparse
from time import strftime, gmtime


def url_normalize(path):
    if path.startswith("."):
        path = "/" + path
    while "../" in path:
        p1 = path.find("/..")
        p2 = path.rfind("/", 0, p1)
        if p2 != -1:
            path = path[:p2] + path[p1+3:]
        else:
            path = path.replace("/..", "", 1)
    path = path.replace("/./", "/")
    path = path.replace("/.", "")
    return path


class FileProducer(object):

    def __init__(self, file, chunk_size=4096):
        self.file = file
        self.chunk_size = chunk_size

    def more(self):
        if self.file:
            data = self.file.read(self.chunk_size)
            if data:
                return data
            self.file.close()
            self.file = None
        return ""


class AsyncServer(asyncore.dispatcher):

    def __init__(self, host="127.0.0.1", port=9000):
        pass

    def handle_accept(self):
        pass

    def serve_forever(self):
        pass


class AsyncHTTPRequestHandler(asynchat.async_chat):

    def __init__(self, sock):
        pass

    def collect_incoming_data(self, data):
        pass

    def found_terminator(self):
        pass

    def parse_request(self):
        pass

    def parse_headers(self):
        pass

    def handle_request(self):
        pass

    def send_header(self, keyword, value):
        pass

    def send_error(self, code, message=None):
        pass

    def send_response(self, code, message=None):
        pass

    def end_headers(self):
        pass

    def date_time_string(self):
        pass

    def send_head(self):
        pass

    def translate_path(self, path):
        pass

    def do_GET(self):
        pass

    def do_HEAD(self):
        pass

    responses = {
        200: ('OK', 'Request fulfilled, document follows'),
        400: ('Bad Request',
            'Bad request syntax or unsupported method'),
        403: ('Forbidden',
            'Request forbidden -- authorization will not help'),
        404: ('Not Found', 'Nothing matches the given URI'),
        405: ('Method Not Allowed',
            'Specified method is invalid for this resource.'),
    }


def parse_args():
    parser = argparse.ArgumentParser("Simple asynchronous web-server")
    parser.add_argument("--host", dest="host", default="127.0.0.1")
    parser.add_argument("--port", dest="port", type=int, default=9000)
    parser.add_argument("--log", dest="loglevel", default="info")
    parser.add_argument("--logfile", dest="logfile", default=None)
    parser.add_argument("-w", dest="nworkers", type=int, default=1)
    parser.add_argument("-r", dest="document_root", default=".")
    return parser.parse_args()

def run():
    server = AsyncServer(host=args.host, port=args.port)
    server.serve_forever()

if __name__ == "__main__":
    args = parse_args()

    logging.basicConfig(
        filename=args.logfile,
        level=getattr(logging, args.loglevel.upper()),
        format="%(name)s: %(process)d %(message)s")
    log = logging.getLogger(__name__)

    DOCUMENT_ROOT = args.document_root
    for _ in xrange(args.nworkers):
        p = multiprocessing.Process(target=run)
        p.start()
```

### Асинхронный WSGI-Server

```py
class AsyncWSGIServer(httpd.AsyncServer):

    def set_app(self, application):
        self.application = application

    def get_app(self):
        return self.application


class AsyncWSGIRequestHandler(httpd.AsyncHTTPRequestHandler):

    def get_environ(self):
        pass

    def start_response(self, status, response_headers, exc_info=None):
        pass

    def handle_request(self):
        pass

    def finish_response(self, result):
        pass
```

```py
import falcon
import json


class QuoteResource:

    def on_get(self, req, resp):
        quote = {
            'quote': 'I\'ve always been more interested in the future than i    n the past.',
            'author': 'Grace Hopper'
        }
        resp.body = json.dumps(quote)

    def on_post(self, req, resp):
        pass

api = falcon.API()
api.add_route('/quote', QuoteResource())
```

### uWSGI



