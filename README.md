# Neoannophobia
Challenge for ICTF




# The challenge
![challenge_image](https://user-images.githubusercontent.com/98354876/179836208-1f0cab80-1cec-4dbb-83a8-a265c7ae545f.png)


 The challenge was a 100 point misc challenge. A super fun challenge and defintely my favorite.
 So, lets jump into it.
 
 # Initial prompt
 After connecting to the service, we are given this prompt.
 

![init](https://user-images.githubusercontent.com/98354876/179930327-660b98fd-273e-48f9-9c5c-7c00b9c1ad5b.png)


Essentially, our goal is to type out December 31 first.
The rules are as follows:
A random date is chosen in January as the starting date
We cannot go backwards in time, only forward.
We have to match one of two things: 

One, The month that the previous player chose, then we can choose any date in that month thats in the future.


Two, The day that the previous player chose, then we can choose any month in the future with that day.
We cannot choose the same date the previous player chose


Finally, to get the flag, we need to win 100 times against the computer.
Ok, cool, we now know the rules and goal. Lets begin

# Messing around

Well, without much info, lets just begin messing around and see how this machine played.
Since our first prompt was Janaury 31, lets try just entering December 31. We are keeping the day, and moving forward in months.


![initialWin](https://user-images.githubusercontent.com/98354876/179929676-d157f123-a9b6-45f9-abba-ca72444c4ada.png)

Well... That was easy. Our first win of many to come.
So, We can add something to our notes; any time the day "31" shows up, we can instantly jump to December 31 and win that round.
Good to know, lets move forward. Our next prompt is January 12. No easy win this time, but lets try winning anyway. Im going to select a random month. Lets say, June.

However, when i went to enter this date, I came back my connection terminated. We can now add something else to our notes: The ability to do this all manually wont be possible.
With no time like the present, lets begin making our script. I like to use pwntools due to its easy to understand utility.
```python
	from pwn import *
	p =remote("neoannophobia.chal.imaginaryctf.org", 1337)
	p.recvuntil(b"\n\n\n")
	print(p.recvline())
	print(p.recvline())
	print(p.recvline())
```
All this script is doing its connecting to the service, recieving all the junk at the start line by line.
Now, out of my curiousity, i wondered if this idea was new, or if this was a solved game.
Looking up "Race to December 31" Results in a website with a winning strategy.
https://mindyourdecisions.com/blog/2016/08/21/the-race-to-december-31-sunday-puzzle/

This suggests that the winning strategy for each day and month we want to choose is day = month + 19.
Doing this for each month, we end up with the following.


1/20 2/21 3/22 4/23 5/24 6/25 7/26 8/27 9/28 10/29 11/30 12/31


There is a small issue, we dont control the first day, so if the computer plays perfectly, we can never win.
Luckily for us, as we saw from our initial prompt, its not exactly smart.

So, if he doesnt play January 20th, we should be able to, thus resulting in our inevitable victory!

# Creating the script
Lets start adding onto our script using this winning strategy. The way I decided to do this was with a key : value dictionary, its simple and allows for us to reference the month given to us, and play the correct winning move every time. We should also add a round counter so we know when to stop

Now our script is the following

```python
	from pwn import *
	cr=1
	hashmap={'January': '20', 'February': '21', 'March': '22', 'April': '23', 'May': '24', 'June': '25', 'July': '26', 'August': '27', 'September': 	'28', 'October': '29', 'November': '30', 'December': '31'}
	p =remote("neoannophobia.chal.imaginaryctf.org", 1337)
	p.recvuntil(b"\n\n\n")
	print(p.recvline())
	print(p.recvline())
	print(p.recvline())
```
Cool, we have a dictionary we can reference with these winning moves.
Lets start automating our first input, as well as recieving the response.
We know that at the beginning of this prompt, 3 lines are printed before the new month and day are given.

So we can just use "p.recvline()" to get all 3 of those lines, then we know the next line is the date.
Something like this would work. However, since we just want to enter the dates and recieve this first date, we just need one "p.recv()" which will contain the date. We can worry about grabbing those extra lines later. Also, assuming we need to do this 100 times, lets put it into a while loop.

The reason *why* i am setting cr to 101 and not 100, is because our cr variable is set to 1 at the start. We need 100 rounds after that, whats 100 + 1? 101
```python
	while cr !=101:
		string = p.recv()
```
Now, p.recv() bascially selects every single bit of data it can.

We know that it contains the month and date, so lets split on a space and get those into variables.
```python
	month,day=string.split()
```
Cool now this should give us the correct date !

![firsterror](https://user-images.githubusercontent.com/98354876/179929341-de24313c-5706-453d-8fbf-8e2e5ff6dd64.png)

...or not?? Too many values? But i thought it was just the month and day?
Well, lets print out the string variable so we know what it actual contains.

![ahamoment](https://user-images.githubusercontent.com/98354876/179929369-5ed7fd69-7b3c-4887-bfd4-0114a71f1e58.png)


Aha! So the reason it didnt work was because there is the ">" string as well. Well, no issue, we can just assign a throwaway variable, ill just call it tmp.
```python
	month,day,tmp=string.split()
```
And now our split works great, we have a month and day separately 

So! Lets send the line "January 20" After we recieve the date and then recieve the next date! (Remember, if we send a date and dont win, its just one line that is returned to us)
To send a line with pwntools, we can use

	p.sendline(DATA)
or in our case
```python
	p.sendline(b"January 20")
	print(p.recvline())
```
Adding in some debug code just so we know whats happening, heres our current script.
```python
	from pwn import *
	cr=1
	hashmap={'January': '20', 'February': '21', 'March': '22', 'April': '23', 'May': '24', 'June': '25', 'July': '26', 'August': '27', 'September': 	'28', 'October': '29', 'November': '30', 'December': '31'}
	p =remote("neoannophobia.chal.imaginaryctf.org", 1337)
	p.recvuntil(b"\n\n\n")
	print(p.recvline())
	print(p.recvline())
	p.recvline()

	while cr!=101:
		string=p.recv()
		month,day,tmp=string.split()
		print("\n")
		print("What we are recieving "+month.decode("utf-8"), day.decode())
		p.sendline(b"January 20")
		response=p.recv()
		print(response)
```

![firstsend](https://user-images.githubusercontent.com/98354876/179929427-7e2c7e32-10c6-47fc-a8b4-34dfb2c7922e.png)


And we get a response!
Now that we can effectively interact with the service, lets try to use our dictionary to auto-submit these winning moves.
To do that, we need to reference what the month we recieved was and send that day. IE: If we recieve January, send January 20, if we recieve February, send February 21, etc etc.
However, one issue comes to mind. Remember the rule that we cant go back in the past? Well, we know the machine can response with *any* day from January, including dates above January 20th. So what now? Is it lost, over, doomed?
No! We still have hope!

If you look at the winning moves, they cover all dates that are above the 20th! Remember the rule that we can use the same day, different month? Why not just jump ahead to the winning month depending on what day they choose. IE: If they choose January 30th, we can jump to the corresponding month, that being November. We send November 30 and we are winning again.

Now how might we implement this? Well we need to get the inverse of the key:value, meaning we supply the value and get the key. I googled for a bit, and found this one liner that works
```python
	month=list(hashmap.keys())[list(hashmap.values()).index(day)]
```
This sets the month we want from the day we recieve. perfect!
We also need to deal with if the program selects January 20th as its first input. For that, we can just send January 21 and it should be work 95% of the time.

So lets add a few if statements to take care of the responses and send off our month and day with
```python
	p.sendline(month + b" " + day)
```
*(Note) When recieving data, it is in bytes format. To convert it back to a string, do VARNAME.decode("utf-8").
you also need to re-encode it before sending it off using VARNAME.encode()* 
```python
	while cr!=101:
		string=p.recv() #Recieve the date
		month,day,tmp=string.split() #split it
		print("\n")
		print("What we are recieving "+month.decode("utf-8"), day.decode()) #Some debug statements
		day=day.decode("utf-8")
		month=month.decode("utf-8") #Decoding them so we can use them in our if statements
		day=int(day) #Converting day to an int so we can use < and >
		if day > 20 and month == "January": #If the month is January and the day is above 20, convert day to string and get the inverse.
			day=str(day)
			month=list(hashmap.keys())[list(hashmap.values()).index(day)]
		elif month=="January" and day==20: #If its January 20th, send January 21st
			month="January"
			day=21
		elif month=="January": #Else, meaning its not above or equal to 20, send 20.
			day=20
		day=str(day).encode() #convert both of these back into bytes
		month=month.encode()
		print("What we are sending "+month.decode()+" "+day.decode()+ "\nOn round "+str(cr)) #Debug code
		p.sendline(month+b" "+day) #Send off our current month and date
```
Note we are only checking the month January, because after January, they are stuck, and we have all the winning moves. Just match up whatever day they choose with the month in our key value, and eventually it leads to December 31!


![twocorrect](https://user-images.githubusercontent.com/98354876/179929466-c86c7744-4c42-4a5e-ae72-ad818c2fab2e.png)


Hooray! We got two responses now, and we sent the correct month and date for the first response! So, what now? Well, now we need to handle the other months cases. So if the other month (July in this case) Sends a day, we need to take the inverse of it and send off the correct month!
If it was implemented now, we wouldve gotten a win, as it would looked up December 31! Lets add an "else" statement that takes the inverse and sets the day and month variable, ready to be sent off!
```python
		else:
		   day=str(day)
		   month=list(hashmap.keys())[list(hashmap.values()).index(day)]
```
Keep in mind if we win, we need to recieve 4 lines, and then the date because of the new round. Thats not so hard to do, we can just check if the month and day we are about to send off is December 31, and if it is, we can send it, recieve 4 lines, and then end the if case, as our while loop will recieve the new date!  If it is not December 31, we can just send it normally with an Else Statement. Lets get to implementing it.
```python
	month=month.encode() #Encoding both month and day
	day=str(day).encode()
	if month==b'December' and day == b'31':
		print("What we are sending December 31\nOn round "+str(cr)) #Debug code
		p.sendline(b"December 31")
		p.recvline()
		p.recvline()
		p.recvline()
		p.recvline()
		cr+=1
	else:
		print("What we are sending "+month.decode()+" "+day.decode()+ "\nOn round "+str(cr)) #Debug code
		p.sendline(month+b" "+day) #Send off our current month and date
		
		
```
We run it, however..


![almost](https://user-images.githubusercontent.com/98354876/179929497-f25f3f72-d37e-431c-821d-94d7adb26026.png)


What if the machine goes further in months, copying our day? Then this doesnt work. So! Lets try to fix this. The way i did this was creating a tempmonth variable which copied the current month we were about to send off.
Then, when the loop starts over, i check to see if that tempmonth is equal to the month we *just* recieved. If it is, we use the inverse again, using the day we recieved to get the month we should send.
We also need an "else:" statement. We can make our else simply set the day equal to the winning day from the month we just recieved.
IE: If we recieved March, we set day to 22. adding this to our if statement 
```python
	elif month==tempmonth:
		day=str(day)
		month=list(hashmap.keys())[list(hashmap.values()).index(day)]
	else:
		day=str(day)
		day=hashmap[month]
```
# Finale

Finally, after all this time.We have completed the successful script. Surely we can get the flag now, i mean, we automated it, it can complete 100 rounds. lets go!

![we-lose2](https://user-images.githubusercontent.com/98354876/179924480-ca15bb7b-ff42-41af-9894-2047a8acce41.png)



And our flag ends up being... Error?
So I was stuck on this error for a while, I mean, it should work, right?
But eventually I figured out I had too many "p.recvline()" lines. Well.. lets mitigate that by adding onto our if, else statement at the bottom that determines which month/day we send. We can just add 
```python
	if month==b"December' and day == b"31" and cr == 100:
		p.sendline(month + b" " + day)
		print(p.recvline())
		print(p.recvline())
		break
```
This code took me a bit of fidgeting, but i found out two lines are printed after line 100, so we recieve two. 
One includes the "You won!" text, the other, includes our flag.


![flag](https://user-images.githubusercontent.com/98354876/179929538-9e262fc3-7bae-4ffc-a79b-953fb3e8af30.png)


	ictf{br0ken_gamea_smh_8b1f014a}
Solve script below
-------------------------
```python
	from pwn import *
	cr=1
	hashmap={'January': '20', 'February': '21', 'March': '22', 'April': '23', 'May': '24', 'June': '25', 'July': '26', 'August': '27', 'September': 	'28', 'October': '29', 'November': '30', 'December': '31'}
	p =remote("neoannophobia.chal.imaginaryctf.org", 1337)
	#Recieve all the useless data
	p.recvuntil(b"\n\n\n")
	print(p.recvline())
	print(p.recvline())
	p.recvline()

	while cr!=101:
		string=p.recv() #Recieve the date
		month,day,tmp=string.split() #split it
		print("\n")
		print("What we are recieving "+month.decode("utf-8"), day.decode()) #Some debug statements
		day=day.decode("utf-8")
		month=month.decode("utf-8") #Decoding them so we can use them in our if statements
		day=int(day) #Converting day to an int so we can use < and >
		if day > 20 and month == "January": #If the month is January and the day is above 20, convert day to string and get the inverse.
			day=str(day)
			month=list(hashmap.keys())[list(hashmap.values()).index(day)]
		elif month=="January" and day==20: #If its January 20th, send January 21st
			month="January"
			day=21
		elif month=="January": #Else, meaning its not above or equal to 20, send 20.
			day=20
		elif month==tempmonth:
			day=str(day)
			month=list(hashmap.keys())[list(hashmap.values()).index(day)] #If they stayed in the same month, get the winning month according to the number
		else:
			day=str(day)
			day=hashmap[month] #otherwise, just set the winning day according to what month was recieved

		month=month.encode() #Encode everything
		day=str(day).encode()
		tempmonth=month.decode("utf-8")
		if month==b"December" and day==b"31" and cr==100:
			p.sendline(month+b" "+day)
			print(p.recvline())
			print(p.recvline())
			break
		elif month==b'December' and day == b'31': #if its December 31, send it and recieve those extra lines
			print("What we are sending December 31\nOn round "+str(cr)) #Debug code
			p.sendline(b"December 31")
			p.recvline()
			p.recvline()
			p.recvline()
			p.recvline()
			cr+=1
		else: #If its not December 31, send the month and day we have.
			print("What we are sending "+month.decode()+" "+day.decode()+ "\nOn round "+str(cr)) #Debug code
			p.sendline(month+b" "+day) #Send off our current month and date
		```
