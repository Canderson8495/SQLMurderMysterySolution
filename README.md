# SQL Murder Mystery Solution
https://mystery.knightlab.com/

SELECT * FROM crime_scene_report WHERE type == 'murder' AND date == 20180115
              

20180115	murder	Security footage shows that there were 2 witnesses. The first witness lives at the last house on "Northwestern Dr". The second witness, named Annabel, lives somewhere on "Franklin Ave".	SQL City


So there are two witnesses, let's get their data


SELECT * FROM person WHERE address_street_name == 'Northwestern Dr' ORDER BY address_number DESC

Assuming the highest or lowest number is the last house on the street we can see that it is either 

14887	Morty Schapiro	118009	4919	Northwestern Dr	111564949 < -- this one

Witness #2 

SELECT * FROM person WHERE address_street_name == 'Franklin Ave' AND name LIKE 'Annabel%'

16371	Annabel Miller	490173	103	Franklin Ave	318771143

Let's check out their interviews

SELECT * FROM interview WHERE person_id = 14887 OR person_id == 16371

Morty Schapiro

14887	I heard a gunshot and then saw a man run out. He had a "Get Fit Now Gym" bag. The membership number on the bag started with "48Z". Only gold members have those bags. The man got into a car with a plate that included "H42W".

Annabel Miller

16371	I saw the murder happen, and I recognized the killer from my gym when I was working out last week on January the 9th.

Well this isn't good, but we have a lead. Let's check out who was present on January 9th

X0643	20180109	957	1164
UK1F2	20180109	344	518
XTE42	20180109	486	1124
1AE2H	20180109	461	944
6LSTG	20180109	399	515
7MWHJ	20180109	273	885
GE5Q8	20180109	367	959
48Z7A	20180109	1600	1730
48Z55	20180109	1530	1700
90081	20180109	1600	1700

We do have a lead, but it could be a red herrring.

SELECT * FROM get_fit_now_check_in WHERE check_in_date == 20180109 AND membership_id LIKE '48Z%'

48Z7A	20180109	1600	1730
48Z55	20180109	1530	1700

Let's follow these guys and see what we can find...

SELECT * FROM get_fit_now_check_in WHERE check_in_date == 20180109 AND check_in_date == 20180115 AND membership_id LIKE '48Z%'

NOTHIGN

neither of these guys seem to have shown up on the murder date, but if you were going to murder someone, you should not check in. No worries, let's check out to see if any of these guys are gold members. 

Let's get fun and try a join

SELECT * from get_fit_now_check_in JOIN get_fit_now_member ON get_fit_now_check_in.membership_id = get_fit_now_member.id WHERE check_in_date == '20180109' AND membership_id LIKE '48Z%'

God that was annoying, but now we have more information
membership_id	check_in_date	check_in_time	check_out_time	id	person_id	name	membership_start_date	membership_status
48Z7A	20180109	1600	1730	48Z7A	28819	Joe Germuska	20160305	gold
48Z55	20180109	1530	1700	48Z55	67318	Jeremy Bowers	20160101	gold

I was hoping to see a difference in their membership status, but we still do have some good information here. Two potential suspects.

Witness #1 mentioned a license plate, so let's see if anyone of them has that license plate.

The man got into a car with a plate that included "H42W"

Okay, so let's check driver_licenses for these people.

SELECT * from drivers_license WHERE plate_number LIKE '%H42W%' and (id == 28819 OR id == 67318)

No results. hmm. Maybe the car isn't registered under them.

SELECT * from drivers_license WHERE plate_number LIKE '%H42W%'

id	age	height	eye_color	hair_color	gender	plate_number	car_make	car_model
183779	21	65	blue	blonde	female	H42W0X	Toyota	Prius
423327	30	70	brown	brown	male	0H42W2	Chevrolet	Spark LS
664760	21	71	black	black	male	4H42WR	Nissan	Altima

Let's get these peoples names

SELECT * from drivers_license JOIN person ON drivers_license.id = person.license_id WHERE plate_number LIKE '%H42W%'

id	age	height	eye_color	hair_color	gender	plate_number	car_make	car_model	id	name	license_id	address_number	address_street_name	ssn
664760	21	71	black	black	male	4H42WR	Nissan	Altima	51739	Tushar Chandra	664760	312	Phi St	137882671
423327	30	70	brown	brown	male	0H42W2	Chevrolet	Spark LS	67318	Jeremy Bowers	423327	530	Washington Pl, Apt 3A	871539279
183779	21	65	blue	blonde	female	H42W0X	Toyota	Prius	78193	Maxine Whitely	183779	110	Fisk Rd	137882671

So Jeremy Bowers car was seen leaving the scene. Jeremy Bowers was also in the the gym a week ago.

It's looking alot like Jeremy Bowers, but we must be sure. We can't put an innocent man in jail!


Let's check out what's going on with everyone of the day of the crime
Monty Shapiro
SELECT * FROM facebook_event_checkin WHERE person_id == 14887

person_id	event_id	event_name	date
14887	4719	The Funky Grooves Tour	20180115

Annabel and Monty were both soind the funky grooves tour. Maybe some music group?

SELECT * FROM facebook_event_checkin WHERE person_id == 67318

What is going on. Jeremy Bowers was also at the Funky Grooves tour. 

Do we have an interview from Jeremy?

Wait a minute, we have a interview from Jeremy 
I was hired by a woman with a lot of money. I don't know her name but I know she's around 5'5" (65") or 5'7" (67"). She has red hair and she drives a Tesla Model S. I know that she attended the SQL Symphony Concert 3 times in December 2017. 

This is confusing!

Let's look into this women to see if she matches anything we've seen so far.
SELECT * from drivers_license JOIN person ON drivers_license.id = person.license_id WHERE car_make == 'Tesla' AND hair_color == 'red'



SELECT * from drivers_license LEFT JOIN person ON drivers_license.id = person.license_id JOIN facebook_event_checkin ON person.id = facebook_event_checkin.person_id WHERE drivers_license.car_make == 'Tesla' AND drivers_license.hair_color == 'red' 


id	age	height	eye_color	hair_color	gender	plate_number	car_make	car_model	id	name			license_id	address_number	address_street_name	ssn		person_id	event_id	event_name		date
202298	68	66	green		red		female	500123		Tesla		Model S		99716	Miranda Priestly	202298		1883		Golden Ave		987756388	99716		1143		SQL Symphony Concert	20171206
202298	68	66	green	red	female	500123	Tesla	Model S	99716	Miranda Priestly	202298	1883	Golden Ave	987756388	99716	1143	SQL Symphony Concert	20171212
202298	68	66	green	red	female	500123	Tesla	Model S	99716	Miranda Priestly	202298	1883	Golden Ave	987756388	99716	1143	SQL Symphony Concert	20171229
736081	79	69	brown	red	male	GCAQ6Y	Tesla	Model S	57410	Cletus Zoeller	736081	2987	Kingham Way	924648898	57410	1649	knows what it is. 	20170701
736081	79	69	brown	red	male	GCAQ6Y	Tesla	Model S	57410	Cletus Zoeller	736081	2987	Kingham Way	924648898	57410	3841	harder. 	20170610
736081	79	69	brown	red	male	GCAQ6Y	Tesla	Model S	57410	Cletus Zoeller	736081	2987	Kingham Way	924648898	57410	6505	God did not create the world in 7 days; he screwed around for 6 days 	20171119


OOOH, we have a another lead. Miranda Priestly 99716. She isn't a member...


I think it's time to submit. We have a killer (Jeremy bowers) who was hired by a rich women (Miranda Priestly)

And we are correct.
