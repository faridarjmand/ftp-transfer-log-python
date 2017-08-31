## Created By.Farid Arjmand ##

### This Script use for Auto Transfer Log with FTP


In Python


```sh
#!/usr/bin/python

import os
import sys
import gzip
import socket
import shutil
import hashlib
import difflib
from ftplib import FTP
from time import strftime

##############################
########## Variable ##########
##############################

host='127.0.0.1'
username='testuser'
password='testpass'
port= 21

format = ".log"
date = strftime("%Y-%m-%d-%H:%M")

listlog = []
ftplist = []

###############################
########## Functions ##########
###############################

def checksum(file, txt):
	temp = sys.stdout
	sys.stdout = open(txt, 'a')
	hash = hashlib.md5(open(file, 'rb').read()).hexdigest()
	print(file, hash)
	sys.stdout.close()
	sys.stdout = temp


def compress(i):
	global j
	j = []
	j.append(str(i))
	j.append('.gz')
	j = ''.join(j)
	ftplist.append(j)
	infile = open(i, 'rb')
	outfile = gzip.open(j, 'wb')
	outfile.writelines(infile)
	outfile.close()
	infile.close()

def check_ftp_connection():
	sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
	result = sock.connect_ex((host, port))
	if result != 0:
		print ("Can't Connect To Server !!")
		exit()

def ftp_send(j):
	ftp.storbinary('STOR %s' % j, open(j, 'rb'))
	shutil.move(j, date)

def ftp_recive(ii):
	ftp.retrbinary('RETR %s' % ii,  open(ii, 'wb').write)

def ftp_login(date):
	global ftp
	ftp = FTP(host,username,password)
	ftp.mkd(date)
	ftp.cwd(date)

def remove_checksum():
	for check in os.listdir("."):
		if check.startswith("checksum"):
			os.remove(check)

def diff():
	checksum = open('checksum.txt').readlines()
	checksum_ftp = open('checksum_ftp.txt').readlines()
	if checksum != checksum_ftp:
		print ("Data is broken !!")
		diff_checksum()

def diff_checksum():
	diff = difflib.ndiff(open('checksum.txt').readlines(), open('checksum_ftp.txt').readlines())
	try:
		while 1:
			print diff.next(),
	except:
		pass

##############################
############ Main ############
##############################

check_ftp_connection()

for log in os.listdir("."):
	if log.endswith(format):
		listlog.append(os.path.join(log))

os.makedirs(date)
ftp_login(date)

for i in listlog:
	if os.path.isfile(i):
		size = os.stat(i).st_size
		if size != 0:
			compress(i)
			checksum(file=j, txt='checksum.txt')
			ftp_send(j)
		os.remove(i)

ftp.storbinary('STOR checksum.txt', open('checksum.txt', 'rb'))
shutil.copy('checksum.txt', date)

for ii in ftplist:
	ftp_recive(ii)
	checksum(file=ii, txt='checksum_ftp.txt')
	os.remove(ii)

ftp.quit()
diff()
remove_checksum()

##############################
############ END #############
##############################
```
