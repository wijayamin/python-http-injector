#!/usr/bin/python

import socket
import select
import time
import argparse
import json
import logging
import sys

logging.basicConfig(
    filename='/tmp/http-injector.log',
    filemode='w',
    format='%(asctime)s %(message)s',
    level=logging.DEBUG)

# forward trafic to remote proxy
class Forward:
    def __init__(self):
        self.forward = socket.socket(socket.AF_INET, socket.SOCK_STREAM)

    def start(self, host, port):
        try:
            self.forward.connect((host, port))
            return self.forward
        except Exception, e:
            print e
            return False


class TheServer:
    input_list = []
    channel = {}
    channel_ = {}
    request = {}

    # init TheServer Class
    def __init__(self, config, port):

        payload_file = json.load(config)

        payload = payload_file['payload']
        payload = payload.replace('[crlf]', '\r\n')
        payload = payload.replace('[lf]', '\n')
        payload = payload.replace('[cr]', '\r')
        payload = payload.replace('[protocol]','HTTP 1.0')

        self.payload = payload
        self.forward_to = (payload_file['proxy']['host'], payload_file['proxy']['port'])
        self.buffer_size = payload_file['buffer']

        self.server = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
        self.server.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)
        self.server.bind(('0.0.0.0', port))
        self.server.listen(200)

    def on_accept(self):
        forward = Forward().start(self.forward_to[0], self.forward_to[1])
        clientsock, clientaddr = self.server.accept()
        if forward:
            self.input_list.append(clientsock)
            self.input_list.append(forward)
            self.channel[clientsock] = forward
            self.channel[forward] = clientsock
            self.channel_[clientsock] = forward
            self.channel_[forward] = forward
        else:
            print "Proxy tidak merespon",
            print "Closing connection with client side", clientaddr
            clientsock.close()

    def on_close(self):
        self.input_list.remove(self.s)
        self.input_list.remove(self.channel[self.s])

        out = self.channel[self.s]

        self.channel[out].close()
        self.channel_[out].close()

        self.channel[self.s].close()
        self.channel_[self.s].close()

        del self.channel[out]
        del self.channel_[out]

        del self.channel[self.s]
        del self.channel_[self.s]

    def on_execute(self):
        netdata = self.netdata
        if netdata.find("CONNECT") == 0:
            req = netdata.split('HTTP')[0]
            req = req.split(' ')
            host_port = req[1].split(':')

            proto = netdata.split('HTTP')[1]
            print proto.replace('/', '')

            payloads = self.payload
            payloads = payloads.replace('[host_port]', req[1])
            payloads = payloads.replace('[host]', host_port[0])
            payloads = payloads.replace('[port]', host_port[1])
            # print host_port[1]
            if payloads.find('[split]') <> -1:
                pay = payloads.split('[split]')
                self.request[self.channel[self.s]] = pay[1]
                netdata = pay[0]
            else:
                netdata = payloads
            print netdata
        try:
            self.channel[self.s].send(netdata)
        except Exception, e:
            print e

    def on_outbounddata(self):
        netdata = self.netdata
        if netdata.find('HTTP/1.') == 0:
            # print netdata
            if self.payload.find('[split]') <> -1:
                if self.request[self.s] != '':
                    time.sleep(0.5)
                    print self.request[self.s]
                    self.channel_[self.s].send(self.request[self.s])
                    self.request[self.s] = ''
            netdata = 'HTTP/1.1 200 Connection established\r\n\r\n'
        try:
            self.channel[self.s].send(netdata)
        except Exception, e:
            print e

    def main_loop(self):
        self.input_list.append(self.server)
        while 1:
            ss = select.select
            inputready, outputready, exceptready = ss(self.input_list, [], [])

            for self.s in inputready:
                if self.s == self.server:
                    self.on_accept()
                    break
                try:
                    self.netdata = self.s.recv(self.buffer_size)
                except Exception, e:
                    self.netdata =''
                if len(self.netdata) == 0:
                    self.on_close()
                else:
                    if cmp(self.channel[self.s],self.channel_[self.s]):
                        self.on_outbounddata()
                    else:
                        self.on_execute()


# initiate main program
if __name__ == '__main__':
    parser = argparse.ArgumentParser(
        prog='http-injector',
        description='Python Version of HTTP-INJECTOR')
    parser.add_argument(
        'config', metavar='payload',
        type=argparse.FileType('r'),
        help='payload file')
    parser.add_argument(
        '-l', dest='listen', nargs='?', const=1989,
        help='listen port', default=1989)
    args = parser.parse_args()

    server = TheServer(args.config, int(args.listen))
    try:
        server.main_loop()
    except KeyboardInterrupt:
        print "Ctrl C - Stopping server"


