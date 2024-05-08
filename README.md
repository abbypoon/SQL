# This is the solution for the game SQL Murder Mystery 

#To find out who were the witness:

SELECT * FROM "crime_scene_report" 
WHERE type = "murder" AND city = "SQL City" AND date = 20180115

#Answer:

Security footage shows that there were 2 witnesses. The first witness lives at the last house on "Northwestern Dr". The second witness, named Annabel, lives somewhere on "Franklin Ave".

#To find out the first witness:

SELECT * FROM "person" 
WHERE address_street_name = "Northwestern Dr" 
ORDER BY address_number DESC
LIMIT 1

#Answer:

id	name	license_id	address_number	address_street_name	ssn
14887	Morty Schapiro	118009	4919	Northwestern Dr	111564949

#To find out the second witness:

SELECT * FROM "person" 
WHERE address_street_name LIKE "%Franklin Ave%" AND name like "%Annabel%"

#Answer:

id	name	license_id	address_number	address_street_name	ssn
16371	Annabel Miller	490173	103	Franklin Ave	318771143

#First witness’s transcript

SELECT i.person_id, i.transcript
FROM facebook_event_checkin f
JOIN interview i
ON f.person_id = i.person_id
WHERE i.person_id = 14887

#Answer:

person_id	transcript
14887	I heard a gunshot and then saw a man run out. 
He had a "Get Fit Now Gym" bag. The membership number on the bag started with "48Z". 
Only gold members have those bags. The man got into a car with a plate that included "H42W".

#Second witness’s transcript:

SELECT g.name, i.transcript
FROM get_fit_now_member g
JOIN interview i
ON g.person_id = i.person_id
WHERE name = "Morty Schapiro" 
OR name = "Annabel Miller"

#Answer:

name	transcript
Annabel Miller	I saw the murder happen, and I recognized the killer from my gym 
when I was working out last week on January the 9th.

#Who checked in on January the 9th?

SELECT * 
FROM get_fit_now_check_in
WHERE check_in_date = 20180109
AND membership_id LIKE "48Z%"

#Answer:

membership_id	check_in_date	check_in_time	check_out_time
48Z7A	20180109	1600	1730
48Z55	20180109	1530	1700

#Who has gold membership?

SELECT id, person_id, name, membership_status
FROM get_fit_now_check_in 
JOIN get_fit_now_member 
ON membership_id = id
WHERE check_in_date = 20180109
AND membership_id LIKE "48Z%"
AND membership_status = "gold"

#Answer:

id	person_id	name	membership_status
48Z7A	28819	Joe Germuska	gold
48Z55	67318	Jeremy Bowers	gold


#Find out the murder:

SELECT gm.id, gm.person_id, gm.name, membership_status
FROM get_fit_now_check_in gi
JOIN get_fit_now_member gm
ON membership_id = gm.id
JOIN person p
ON gm.person_id = p.id
JOIN drivers_license d
ON license_id = d.id
WHERE check_in_date = 20180109
AND membership_id LIKE "48Z%"
AND membership_status = "gold"
AND plate_number LIKE "%H42W%"

#Answer:

id	person_id	name	membership_status
48Z55	67318	Jeremy Bowers	gold

#To find the transcript of the murder:

SELECT *
FROM interview
WHERE person_id ="67318"

#Anwer:

person_id	transcript
67318	I was hired by a woman with a lot of money. 
I don't know her name but I know she's around 5'5" (65") or 5'7" (67"). 
She has red hair and she drives a Tesla Model S. 
I know that she attended the SQL Symphony Concert 3 times in December 2017.

#To find the real villain behind the crime:

SELECT p.name, fc.person_id, event_name, date
FROM facebook_event_checkin fc
JOIN person p
ON fc.person_id = p.id
JOIN drivers_license d
ON p.license_id=d.id
WHERE event_name = "SQL Symphony Concert"
AND date between 20171201 AND 20171231
AND gender = "female"
AND height BETWEEN 65 AND 67
AND hair_color = "red"
AND car_model = "Model S"

#Answer:

name	person_id	event_name	date
Miranda Priestly	99716	SQL Symphony Concert	20171206
Miranda Priestly	99716	SQL Symphony Concert	20171212
Miranda Priestly	99716	SQL Symphony Concert	20171229

