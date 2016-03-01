# modified-lemonade
# Modification of the lemonade stand simulation

#! /usr/bin/python

from __future__ import division
from __future__ import print_function
from math import log, exp

# Produces the cost to make a glass of lemonade for the day, based upon ingredient cost. 
# Cost increases over time.
def days_cost(day):
	return int(4 + 0.3 * day) # Cents, not dollars

# Gives the number of glasses of lemonade produced for the day. 
# Warmer temperatures increases the number of glasses made. 
# With warmer temperatures there are more thirsty customers and higher chances of lemonade being bought.
def get_glasses(day):
# This is the first line modified. Now get_glasses function returns the integer of the temperature for the day divided by 5.
	return int(days_temperature(day)/5)

# This function was changed make the price of lemonade six plus the number of signs made for the day. 
# Gets the price that a glass of lemonade will be sold at for the day.
# Signs cost money to make, so when signs are made the price of lemonade goes up to avoid losing money. 
def get_price(day, assests, sign_cost):
	return 6 + get_signs(day, assets, sign_cost) # Cents, not dollars
	
# This function was modified to return the number of signs made based upon a series.
# Alternates number of signs produced between zero and one per day. 
def get_signs(day, assets, sign_cost):
	original_signs = 0
	for n in range(days):
		signs_made = 1 - original_signs
		original_signs = signs_made
	return signs_made

# Gives the likelihood that a customer will buy a glass of lemonade. 
# Likelihood decreases as the price per glass of lemonade increases.
def base_demand(price):
	return max(108 - 0.8 * price * price, 0)

# Adds on to the likelihood that a customer will buy lemonade.
# Increases as the number of signs produced for the day increases, since more people are able to know about the lemonade stand. 
def advertising_multiplier(signs):
	return 1 - exp(-signs / 2) / 2

# Produces the temperature outside for the day. 
def days_temperature(day):
	return int(10 + 8.8 * (day % 4)) # Degrees in Celsius, not Fahrenheit

# Increases the likelihood that customers will buy lemonade. 
# Higher temperatures means people are more dehydrated and more likely to buy a lemonade. 
def temperature_multiplier(temperature):
	if temperature >= 15:
		return log(temperature / 15)
	return 0

# Produces a percentage chance of it raining or snowing for the day. 
# If it rains or snows for the day, no lemonade can be sold. 
# Will only snow if precipitation occurs and temperature is less than 5.
def days_chance_of_precipitation(day):
	return int(15 * (day % 5)) / 100  

# Decreases the likelihood that customers will buy lemonade.
# More likely it is to rain the less people are out and about to walk by the lemonade stand.
def precipitation_multiplier(chance_of_precipitation):
	if chance_of_precipitation > 0.5:
		return 0
	return 1 - chance_of_precipitation / 2

# Determines if there will be construction going on in front of the lemonade stand. 
def days_street_work(day):
	if day == 2:
		return 1
	if day == 8:
		return 0.5
	return 0

# Increases the likelihood of lemonade being sold. 
# The construction workers will get thirsty on the job and since the lemonade stand is right there they may buy lemonade.
def street_work_multiplier(street_work):
	exponent = street_work
	if street_work < 1:
		exponent = -exponent
	return exp(exponent)

print('')
print('deterministic-lemonade.py: A Deterministic Ten-Day Lemonade Stand Simulation')
print('(loosely based on the 1979 game "Lemonade Stand" by the Minnesota Educational Computing Consortium)')
print('')
initial_assets = 200 # Cents
assets = initial_assets
sign_cost = 15 # Cents
days = 10
for day in range(days):
# Can not have negative days. 
	if day > 0:
		print('')
	print('Day {day}:'.format(day = day + 1))
	print('')
	print('  You have {assets} cent(s) in assets.'.format(assets = assets))
	cost = days_cost(day)
	print('  At the current going rate for ingredients, a glass of lemonade will cost {cost} cent(s) to make.'.format(cost = cost))
	temperature = days_temperature(day)
	chance_of_precipitation = days_chance_of_precipitation(day)
	print('  The forecast calls for a temperature of {temperature} degrees Celsius ({converted} degrees Farenheit) with a(n) {percent}% chance of precipitation.'.format(temperature = temperature, converted = int(round(32 + 9 * temperature / 5)), percent = int(chance_of_precipitation * 100)))
	street_work = days_street_work(day)
	if street_work > 0.5:
		print('  There is ongoing street work, reducing traffic significantly.  But perhaps the work crew will get thirsty?')
	elif street_work > 0:
		print('  There is ongoing street work, slightly reducing traffic.  But perhaps the work crew will get thirsty?')
	print('')
	original_assets = assets
	glasses = get_glasses(day)
	lemonade_cost = glasses * cost
	assets -= lemonade_cost
	price = get_price(day, assets, sign_cost)
	signs = get_signs(day, assets, sign_cost)
	advertising_cost = signs * sign_cost
	assets -= advertising_cost
	print('')
	advertising_factor = advertising_multiplier(signs)
	temperature_factor = temperature_multiplier(temperature)
	precipitation_factor = precipitation_multiplier(chance_of_precipitation)
	if precipitation_factor <= 0:
		if temperature < 5:
			if glasses > 0:
				print('It snows!  Your sales are ruined.')
			else:
				print('It snows!  You stay warm inside.')
		else:
			if glasses > 0:
				print('It rains!  Your sales are ruined.')
			else:
				print('It rains!  You stay dry inside.')
	street_work_factor = street_work_multiplier(street_work)
	if precipitation_factor > 0 and street_work_factor > 1 and glasses > 3:
		print('Over lunch, the work crew stops by, and everyone gets lemonade!')
	sales = min(int(base_demand(price) * advertising_factor * temperature_factor * precipitation_factor * street_work_factor), glasses)
	income = sales * price
	assets += income
	print('You began the day with {original_assets} cent(s) in assets.'.format(original_assets = original_assets))
	print('You prepared {glasses} glass(es) of lemonade for a total cost of {lemonade_cost} cent(s).'.format(glasses = glasses, lemonade_cost = lemonade_cost))
	print('You prepared {signs} sign(s) for a total cost of {advertising_cost} cent(s).'.format(signs = signs, advertising_cost = advertising_cost))
	print('You sold {sales} glass(es) of lemonade for a total income of {income} cent(s).'.format(sales = sales, income = income))
	print('Your profit for the day was {profit} cent(s).'.format(profit = income - lemonade_cost - advertising_cost))
	print('You ended the day with {assets} cent(s) in assets.'.format(assets = assets))
print('')
# Gives final results of the simulation
print('You finished the simulation with {assets} cent(s) in assets.'.format(assets = assets))
if assets > 0:
	print('Your business growth rate: {percent}% per day.'.format(percent = int(100 * ((assets / initial_assets) ** (1 / days) - 1))))
print('Your business rating:', end = ' ')
if assets < 50:
	print('Dismal!')
elif assets < 150:
	print('Disappointing')
elif assets < 250:
	print('Okay')
elif assets < 300:
	print('Good')
elif assets < 400:
	print('Great!')
elif assets < 500:
	print('Excellent!')
else:
	print('Incredible!')
print('')
