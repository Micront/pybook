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
        read_waiters[clientsocket.fileno()] = (recv_handler, (clientsocket.f    ileno(),))
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
        rlist, wlist, _ = select.select(read_waiters.keys(), write_waiters.k    eys(), [], 60)
    
        for r_fileno in rlist:
            handler, args = read_waiters.pop(r_fileno)
            handler(*args)
        
        for w_fileno in wlist:
            handler, args = write_waiters.pop(w_fileno)
            handler(*args)
    
if __name__ == "__main__":
    main()
```



