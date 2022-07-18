# Neoannophobia
Challenge for ICTF

Solve script below
-------------------------


	from pwn import *
	cr=1
	hashmap={'January': '20', 'February': '21', 'March': '22', 'April': '23', 'May': '24', 'June': '25', 'July': '26', 'August': '27', 'September': '28', 'October': '29', 'November': '30', 'December': '31'}

	p=remote("neoannophobia.chal.imaginaryctf.org", 1337)
	p.recvuntil(b"\n\n\n")
	print(p.recvline())
	print(p.recvline())
	print(p.recvline())
	check=True
	#Getting rid of all starting text
	while cr !=101:
		string=p.recv()
		# if cr==99:
			# p.interactive()
		if b"You won!" in string:
			string.decode()
			print(b"Flag found: "+string)
		month,day,tmp=string.split()
		print("\n")
		print("What we are recieving "+month.decode("utf-8"), day.decode())
		day=day.decode("utf-8")
		month=month.decode("utf-8")
		day=int(day)
		if day > 20 and month == "January":
			day=str(day)
			month=list(hashmap.keys())[list(hashmap.values()).index(day)]
		elif month=="January" and day==20:
			month="January"
			day=21
		elif month=="January":
			day=20
		elif month==tempmonth:
			day=str(day)
			month=list(hashmap.keys())[list(hashmap.values()).index(day)]
		else:
			day=str(day)
			# month=list(hashmap.keys())[list(hashmap.values()).index(day)]
			day=hashmap[month]
			# print("1")
		month=month.encode()
		day=str(day).encode()
		tempmonth=month.decode("utf-8")
		tempday=day.decode("utf-8")

		if tempday==str(day).encode() and tempmonth==month.decode():
			day=int(day)
			day+=1
			day=str(day).encode()
		print("What we are sending "+month.decode()+" "+day.decode()+ "\nOn round "+str(cr))
		if month==b"December" and day==b"31" and cr==100:
			p.sendline(month+b" "+day)
			print(p.recvline)
			print(p.recvline)
			break
		elif month==b"December" and day==b"31":
			p.sendline(month+b" "+day)
			p.recvline()
			p.recvline()
			p.recvline()
			p.recvline()
			cr+=1
		else:
			p.sendline(month+b" "+day)
		# print(p.recv())
