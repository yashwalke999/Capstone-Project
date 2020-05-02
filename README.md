# new
Coursera_Capstone_Final_Project

Introduction and Business Problem¶
Introduction:

The city of Hoboken, NJ is relatively small at ~1 square mile but it is packed with restaurants, night life and amazing people. For people that are new to Hoboken, despite its small geographic size, it can be daunting to figure out what restaurants are worth going to and where they are. For people that used to live in Hoboken or are visiting Hoboken, how do you know what the best places are to get something to eat?
Business Problem:

For this project, I am going to put on my entrepreneur hat and create a simple guide on where to eat based on Foursquare likes, restaurant category and geographic location data for restaurants in Hoboken. I will then cluster these restaurants based on their similarities so that a user can easily determine what type of restaurants are best to eat at based on Foursquare user feedback.
Data Required
For this assignment, I will be utilizing the Foursquare API to pull the following location data on restaurants in Hoboken, NJ:

Venue Name
Venue ID
Venue Location
Venue Category
Count of Likes
Data Acquisition Approach
To acquire the data mentioned above, I will need to do the following:

Get geolocator lat and long coordinates for Hoboken, NJ
Use Foursquare API to get a list of all venues in Hoboken
Get venue name, venue ID, location, category, and likes
Algorithm Used
I will take the gathered data (see above in Data Acquisition Approach and Data Required sections) and will create a k-means clustering algorithm that groups restaurants into 4-5 clusters so that people looking to eat in Hoboken can easily see which restaurants are the best to eat at, what cuisine is available and where in Hoboken they can look to eat.

Data Prep and Pull
We will import our necessary packages and start pulling our data for data prep and usage.

In [1]:
import numpy as np # library to handle data in a vectorized manner

import pandas as pd # library for data analsysis
pd.set_option('display.max_columns', None)
pd.set_option('display.max_rows', None)

import json # library to handle JSON files

!conda install -c conda-forge geopy --yes # uncomment this line if you haven't completed the Foursquare API lab
from geopy.geocoders import Nominatim # convert an address into latitude and longitude values

import requests # library to handle requests
from pandas.io.json import json_normalize # tranform JSON file into a pandas dataframe

# Matplotlib and associated plotting modules
import matplotlib.cm as cm
import matplotlib.colors as colors

# import k-means from clustering stage
from sklearn.cluster import KMeans

!conda install -c conda-forge folium=0.5.0 --yes # uncomment this line if you haven't completed the Foursquare API lab
import folium # map rendering library

#import beautiful soup
from urllib.request import urlopen
from bs4 import BeautifulSoup


print('Libraries imported.')
Solving environment: done


==> WARNING: A newer version of conda exists. <==
  current version: 4.5.11
  latest version: 4.6.3

Please update conda by running

    $ conda update -n base conda



# All requested packages already installed.

Solving environment: done


==> WARNING: A newer version of conda exists. <==
  current version: 4.5.11
  latest version: 4.6.3

Please update conda by running

    $ conda update -n base conda



# All requested packages already installed.

Libraries imported.
Finding the geo data for Hoboken
Let's find the geographic data for Hoboken so we can pull it from FourSquare.

In [2]:
address = 'Hoboken, New Jersey'

geolocator = Nominatim()
location = geolocator.geocode(address)
latitude = location.latitude
longitude = location.longitude
print('The geograpical coordinate of Hoboken are {}, {}.'.format(latitude, longitude))
/anaconda/lib/python3.6/site-packages/ipykernel_launcher.py:3: DeprecationWarning: Using Nominatim with the default "geopy/1.18.1" `user_agent` is strongly discouraged, as it violates Nominatim's ToS https://operations.osmfoundation.org/policies/nominatim/ and may possibly cause 403 and 429 HTTP errors. Please specify a custom `user_agent` with `Nominatim(user_agent="my-application")` or by overriding the default `user_agent`: `geopy.geocoders.options.default_user_agent = "my-application"`. In geopy 2.0 this will become an exception.
  This is separate from the ipykernel package so we can avoid doing imports until
The geograpical coordinate of Hoboken are 40.7433066, -74.0323752.
FourSquare Part 1
Entering in our information into the Foursquare API to access it.

In [3]:
CLIENT_ID = '' # your Foursquare ID
CLIENT_SECRET = '' # your Foursquare Secret
VERSION = '20180605' # Foursquare API version

print('Your credentails:')
print('CLIENT_ID: ' + CLIENT_ID)
print('CLIENT_SECRET:' + CLIENT_SECRET)
Your credentails:
CLIENT_ID: UI1GTXNA5FGGAMAHX13BSP0G1UZVUD5ZRRYMIGZXBLSMQVPG
CLIENT_SECRET:NUGVEFSFR33GPNEDNYYAXBDL0KK2YD1XMFMPH3PLSP3R05RE
FourSquare Part 2
Creating a URL for all of the venues in Hoboken

In [4]:
LIMIT = 100 # limit of number of venues returned by Foursquare API
radius = 500 # define radius

# create URL
url = 'https://api.foursquare.com/v2/venues/explore?&client_id={}&client_secret={}&v={}&ll={},{}&radius={}&limit={}'.format(
    CLIENT_ID, 
    CLIENT_SECRET, 
    VERSION, 
    latitude, 
    longitude, 
    radius, 
    LIMIT)
url
Out[4]:
'https://api.foursquare.com/v2/venues/explore?&client_id=UI1GTXNA5FGGAMAHX13BSP0G1UZVUD5ZRRYMIGZXBLSMQVPG&client_secret=NUGVEFSFR33GPNEDNYYAXBDL0KK2YD1XMFMPH3PLSP3R05RE&v=20180605&ll=40.7433066,-74.0323752&radius=500&limit=100'
FourSquare Part 3
Pulling the JSON for the URL of venues.

In [5]:
results = requests.get(url).json()
results
Out[5]:
{'meta': {'code': 200, 'requestId': '5c5da9526a60717af1a54cf6'},
 'response': {'groups': [{'items': [{'reasons': {'count': 0,
       'items': [{'reasonName': 'globalInteractionReason',
         'summary': 'This spot is popular',
         'type': 'general'}]},
      'referralId': 'e-0-4cdf46dadb125481eb4236ce-0',
      'venue': {'categories': [{'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/building/gym_',
          'suffix': '.png'},
         'id': '4bf58dd8d48988d175941735',
         'name': 'Gym / Fitness Center',
         'pluralName': 'Gyms or Fitness Centers',
         'primary': True,
         'shortName': 'Gym / Fitness'}],
       'id': '4cdf46dadb125481eb4236ce',
       'location': {'address': '603 Willow Ave',
        'cc': 'US',
        'city': 'Hoboken',
        'country': 'United States',
        'distance': 117,
        'formattedAddress': ['603 Willow Ave',
         'Hoboken, NJ 07030',
         'United States'],
        'labeledLatLngs': [{'label': 'display',
          'lat': 40.744356367758414,
          'lng': -74.03256658205021}],
        'lat': 40.744356367758414,
        'lng': -74.03256658205021,
        'postalCode': '07030',
        'state': 'NJ'},
       'name': 'Work It Out-A Fitness Boutique',
       'photos': {'count': 0, 'groups': []}}},
     {'reasons': {'count': 0,
       'items': [{'reasonName': 'globalInteractionReason',
         'summary': 'This spot is popular',
         'type': 'general'}]},
      'referralId': 'e-0-45e9482df964a52075431fe3-1',
      'venue': {'categories': [{'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/nightlife/pub_',
          'suffix': '.png'},
         'id': '4bf58dd8d48988d11b941735',
         'name': 'Pub',
         'pluralName': 'Pubs',
         'primary': True,
         'shortName': 'Pub'}],
       'id': '45e9482df964a52075431fe3',
       'location': {'address': '343 Park Ave',
        'cc': 'US',
        'city': 'Hoboken',
        'country': 'United States',
        'crossStreet': 'btwn 3rd St & 4th St',
        'distance': 189,
        'formattedAddress': ['343 Park Ave (btwn 3rd St & 4th St)',
         'Hoboken, NJ 07030',
         'United States'],
        'labeledLatLngs': [{'label': 'display',
          'lat': 40.741608,
          'lng': -74.032304}],
        'lat': 40.741608,
        'lng': -74.032304,
        'postalCode': '07030',
        'state': 'NJ'},
       'name': "Onieal's Restaurant & Bar",
       'photos': {'count': 0, 'groups': []}}},
     {'reasons': {'count': 0,
       'items': [{'reasonName': 'globalInteractionReason',
         'summary': 'This spot is popular',
         'type': 'general'}]},
      'referralId': 'e-0-4a4e740ff964a5207bae1fe3-2',
      'venue': {'categories': [{'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/food/bakery_',
          'suffix': '.png'},
         'id': '4bf58dd8d48988d16a941735',
         'name': 'Bakery',
         'pluralName': 'Bakeries',
         'primary': True,
         'shortName': 'Bakery'}],
       'id': '4a4e740ff964a5207bae1fe3',
       'location': {'address': '343 Garden St',
        'cc': 'US',
        'city': 'Hoboken',
        'country': 'United States',
        'crossStreet': '4th St',
        'distance': 200,
        'formattedAddress': ['343 Garden St (4th St)',
         'Hoboken, NJ 07030',
         'United States'],
        'labeledLatLngs': [{'label': 'display',
          'lat': 40.741623250618346,
          'lng': -74.03152338274533}],
        'lat': 40.741623250618346,
        'lng': -74.03152338274533,
        'postalCode': '07030',
        'state': 'NJ'},
       'name': 'Sweet',
       'photos': {'count': 0, 'groups': []}}},
     {'reasons': {'count': 0,
       'items': [{'reasonName': 'globalInteractionReason',
         'summary': 'This spot is popular',
         'type': 'general'}]},
      'referralId': 'e-0-49e9e49df964a5200a661fe3-3',
      'venue': {'categories': [{'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/parks_outdoors/park_',
          'suffix': '.png'},
         'id': '4bf58dd8d48988d163941735',
         'name': 'Park',
         'pluralName': 'Parks',
         'primary': True,
         'shortName': 'Park'}],
       'id': '49e9e49df964a5200a661fe3',
       'location': {'address': '401 Willow Ave.',
        'cc': 'US',
        'city': 'Hoboken',
        'country': 'United States',
        'crossStreet': '4th St.',
        'distance': 129,
        'formattedAddress': ['401 Willow Ave. (4th St.)',
         'Hoboken, NJ 07030',
         'United States'],
        'labeledLatLngs': [{'label': 'display',
          'lat': 40.74215230390258,
          'lng': -74.03223037719727}],
        'lat': 40.74215230390258,
        'lng': -74.03223037719727,
        'postalCode': '07030',
        'state': 'NJ'},
       'name': 'Church Square Park',
       'photos': {'count': 0, 'groups': []}}},
     {'reasons': {'count': 0,
       'items': [{'reasonName': 'globalInteractionReason',
         'summary': 'This spot is popular',
         'type': 'general'}]},
      'referralId': 'e-0-56d3b920498ec4e1c67c0907-4',
      'venue': {'categories': [{'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/nightlife/cocktails_',
          'suffix': '.png'},
         'id': '4bf58dd8d48988d11e941735',
         'name': 'Cocktail Bar',
         'pluralName': 'Cocktail Bars',
         'primary': True,
         'shortName': 'Cocktail'}],
       'id': '56d3b920498ec4e1c67c0907',
       'location': {'address': '500 Grand St',
        'cc': 'US',
        'city': 'Hoboken',
        'country': 'United States',
        'crossStreet': '5th St',
        'distance': 229,
        'formattedAddress': ['500 Grand St (5th St)',
         'Hoboken, NJ 07030',
         'United States'],
        'labeledLatLngs': [{'label': 'display',
          'lat': 40.743209219791716,
          'lng': -74.03509903426071}],
        'lat': 40.743209219791716,
        'lng': -74.03509903426071,
        'postalCode': '07030',
        'state': 'NJ'},
       'name': 'Grand Vin',
       'photos': {'count': 0, 'groups': []}}},
     {'reasons': {'count': 0,
       'items': [{'reasonName': 'globalInteractionReason',
         'summary': 'This spot is popular',
         'type': 'general'}]},
      'referralId': 'e-0-4a1098a2f964a520e3761fe3-5',
      'venue': {'categories': [{'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/food/deli_',
          'suffix': '.png'},
         'id': '4bf58dd8d48988d146941735',
         'name': 'Deli / Bodega',
         'pluralName': 'Delis / Bodegas',
         'primary': True,
         'shortName': 'Deli / Bodega'}],
       'id': '4a1098a2f964a520e3761fe3',
       'location': {'address': '414 Adams St',
        'cc': 'US',
        'city': 'Hoboken',
        'country': 'United States',
        'crossStreet': 'btwn 4th & 5th St',
        'distance': 306,
        'formattedAddress': ['414 Adams St (btwn 4th & 5th St)',
         'Hoboken, NJ 07030',
         'United States'],
        'labeledLatLngs': [{'label': 'display',
          'lat': 40.74299542283236,
          'lng': -74.03598081640737}],
        'lat': 40.74299542283236,
        'lng': -74.03598081640737,
        'postalCode': '07030',
        'state': 'NJ'},
       'name': "Fiore's Deli",
       'photos': {'count': 0, 'groups': []}}},
     {'reasons': {'count': 0,
       'items': [{'reasonName': 'globalInteractionReason',
         'summary': 'This spot is popular',
         'type': 'general'}]},
      'referralId': 'e-0-56daf06fcd107605ef3d86ea-6',
      'venue': {'categories': [{'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/food/bagels_',
          'suffix': '.png'},
         'id': '4bf58dd8d48988d179941735',
         'name': 'Bagel Shop',
         'pluralName': 'Bagel Shops',
         'primary': True,
         'shortName': 'Bagels'}],
       'delivery': {'id': '333353',
        'provider': {'icon': {'name': '/delivery_provider_grubhub_20180129.png',
          'prefix': 'https://fastly.4sqi.net/img/general/cap/',
          'sizes': [40, 50]},
         'name': 'grubhub'},
        'url': 'https://www.grubhub.com/restaurant/obagel-600-washington-st-hoboken/333353?affiliate=1131&utm_source=foursquare-affiliate-network&utm_medium=affiliate&utm_campaign=1131&utm_content=333353'},
       'id': '56daf06fcd107605ef3d86ea',
       'location': {'address': '600 Washington St',
        'cc': 'US',
        'city': 'Hoboken',
        'country': 'United States',
        'crossStreet': 'Sixth Street',
        'distance': 272,
        'formattedAddress': ['600 Washington St (Sixth Street)',
         'Hoboken, NJ 07030',
         'United States'],
        'labeledLatLngs': [{'label': 'display',
          'lat': 40.743603269894244,
          'lng': -74.02917265892027}],
        'lat': 40.743603269894244,
        'lng': -74.02917265892027,
        'postalCode': '07030',
        'state': 'NJ'},
       'name': "O'Bagel",
       'photos': {'count': 0, 'groups': []}}},
     {'reasons': {'count': 0,
       'items': [{'reasonName': 'globalInteractionReason',
         'summary': 'This spot is popular',
         'type': 'general'}]},
      'referralId': 'e-0-4dbc9859f7b1ab37dd636d12-7',
      'venue': {'categories': [{'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/food/sushi_',
          'suffix': '.png'},
         'id': '4bf58dd8d48988d1d2941735',
         'name': 'Sushi Restaurant',
         'pluralName': 'Sushi Restaurants',
         'primary': True,
         'shortName': 'Sushi'}],
       'delivery': {'id': '296370',
        'provider': {'icon': {'name': '/delivery_provider_grubhub_20180129.png',
          'prefix': 'https://fastly.4sqi.net/img/general/cap/',
          'sizes': [40, 50]},
         'name': 'grubhub'},
        'url': 'https://www.grubhub.com/restaurant/ayame-hibachi--sushi-526-washington-st-hoboken/296370?affiliate=1131&utm_source=foursquare-affiliate-network&utm_medium=affiliate&utm_campaign=1131&utm_content=296370'},
       'id': '4dbc9859f7b1ab37dd636d12',
       'location': {'address': '526 Washington St',
        'cc': 'US',
        'city': 'Hoboken',
        'country': 'United States',
        'crossStreet': 'at 5th St',
        'distance': 267,
        'formattedAddress': ['526 Washington St (at 5th St)',
         'Hoboken, NJ 07030',
         'United States'],
        'labeledLatLngs': [{'label': 'display',
          'lat': 40.74310466720745,
          'lng': -74.02921272759819}],
        'lat': 40.74310466720745,
        'lng': -74.02921272759819,
        'postalCode': '07030',
        'state': 'NJ'},
       'name': 'Ayame Hibachi & Sushi',
       'photos': {'count': 0, 'groups': []}}},
     {'reasons': {'count': 0,
       'items': [{'reasonName': 'globalInteractionReason',
         'summary': 'This spot is popular',
         'type': 'general'}]},
      'referralId': 'e-0-57168865498e9517f09fa03d-8',
      'venue': {'categories': [{'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/food/bubble_',
          'suffix': '.png'},
         'id': '52e81612bcbc57f1066b7a0c',
         'name': 'Bubble Tea Shop',
         'pluralName': 'Bubble Tea Shops',
         'primary': True,
         'shortName': 'Bubble Tea'}],
       'delivery': {'id': '446729',
        'provider': {'icon': {'name': '/delivery_provider_grubhub_20180129.png',
          'prefix': 'https://fastly.4sqi.net/img/general/cap/',
          'sizes': [40, 50]},
         'name': 'grubhub'},
        'url': 'https://www.grubhub.com/restaurant/kung-fu-tea-536-washington-st-hoboken/446729?affiliate=1131&utm_source=foursquare-affiliate-network&utm_medium=affiliate&utm_campaign=1131&utm_content=446729'},
       'id': '57168865498e9517f09fa03d',
       'location': {'address': '536 Washington St',
        'cc': 'US',
        'city': 'Hoboken',
        'country': 'United States',
        'distance': 246,
        'formattedAddress': ['536 Washington St',
         'Hoboken, NJ 07030',
         'United States'],
        'labeledLatLngs': [{'label': 'display',
          'lat': 40.7433748,
          'lng': -74.02945}],
        'lat': 40.7433748,
        'lng': -74.02945,
        'postalCode': '07030',
        'state': 'NJ'},
       'name': 'Kung Fu Tea',
       'photos': {'count': 0, 'groups': []}}},
     {'reasons': {'count': 0,
       'items': [{'reasonName': 'globalInteractionReason',
         'summary': 'This spot is popular',
         'type': 'general'}]},
      'referralId': 'e-0-51f1c7e1498e7425c21efab6-9',
      'venue': {'categories': [{'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/shops/apparel_women_',
          'suffix': '.png'},
         'id': '4bf58dd8d48988d108951735',
         'name': "Women's Store",
         'pluralName': "Women's Stores",
         'primary': True,
         'shortName': "Women's Store"}],
       'id': '51f1c7e1498e7425c21efab6',
       'location': {'address': '412 Washington St',
        'cc': 'US',
        'city': 'Hoboken',
        'country': 'United States',
        'crossStreet': 'Btw 4th & 5th St.',
        'distance': 281,
        'formattedAddress': ['412 Washington St (Btw 4th & 5th St.)',
         'Hoboken, NJ 07030',
         'United States'],
        'labeledLatLngs': [{'label': 'display',
          'lat': 40.74183785229468,
          'lng': -74.02966221343871}],
        'lat': 40.74183785229468,
        'lng': -74.02966221343871,
        'postalCode': '07030',
        'state': 'NJ'},
       'name': 'Anthropologie',
       'photos': {'count': 0, 'groups': []}}},
     {'reasons': {'count': 0,
       'items': [{'reasonName': 'globalInteractionReason',
         'summary': 'This spot is popular',
         'type': 'general'}]},
      'referralId': 'e-0-582dfc9565be5809f6a964ed-10',
      'venue': {'categories': [{'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/food/indian_',
          'suffix': '.png'},
         'id': '4bf58dd8d48988d10f941735',
         'name': 'Indian Restaurant',
         'pluralName': 'Indian Restaurants',
         'primary': True,
         'shortName': 'Indian'}],
       'delivery': {'id': '261352',
        'provider': {'icon': {'name': '/delivery_provider_grubhub_20180129.png',
          'prefix': 'https://fastly.4sqi.net/img/general/cap/',
          'sizes': [40, 50]},
         'name': 'grubhub'},
        'url': 'https://www.grubhub.com/restaurant/karma-kafe-505-washington-st-hoboken/261352?affiliate=1131&utm_source=foursquare-affiliate-network&utm_medium=affiliate&utm_campaign=1131&utm_content=261352'},
       'id': '582dfc9565be5809f6a964ed',
       'location': {'address': '505 Washington St',
        'cc': 'US',
        'city': 'Hoboken',
        'country': 'United States',
        'distance': 273,
        'formattedAddress': ['505 Washington St',
         'Hoboken, NJ 07030',
         'United States'],
        'labeledLatLngs': [{'label': 'display',
          'lat': 40.74237277495878,
          'lng': -74.02937643368226}],
        'lat': 40.74237277495878,
        'lng': -74.02937643368226,
        'postalCode': '07030',
        'state': 'NJ'},
       'name': 'Karma Kafe',
       'photos': {'count': 0, 'groups': []}}},
     {'reasons': {'count': 0,
       'items': [{'reasonName': 'globalInteractionReason',
         'summary': 'This spot is popular',
         'type': 'general'}]},
      'referralId': 'e-0-4b660130f964a520ce0d2be3-11',
      'venue': {'categories': [{'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/food/bakery_',
          'suffix': '.png'},
         'id': '4bf58dd8d48988d16a941735',
         'name': 'Bakery',
         'pluralName': 'Bakeries',
         'primary': True,
         'shortName': 'Bakery'}],
       'id': '4b660130f964a520ce0d2be3',
       'location': {'address': '506 Grand St',
        'cc': 'US',
        'city': 'Hoboken',
        'country': 'United States',
        'crossStreet': 'btw 5th & 6th St',
        'distance': 207,
        'formattedAddress': ['506 Grand St (btw 5th & 6th St)',
         'Hoboken, NJ 07030',
         'United States'],
        'labeledLatLngs': [{'label': 'display',
          'lat': 40.743386524526805,
          'lng': -74.03483595097866}],
        'lat': 40.743386524526805,
        'lng': -74.03483595097866,
        'postalCode': '07030',
        'state': 'NJ'},
       'name': "Dom's Bakery",
       'photos': {'count': 0, 'groups': []}}},
     {'reasons': {'count': 0,
       'items': [{'reasonName': 'globalInteractionReason',
         'summary': 'This spot is popular',
         'type': 'general'}]},
      'referralId': 'e-0-4b784457f964a520cdc02ee3-12',
      'venue': {'categories': [{'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/shops/sports_outdoors_',
          'suffix': '.png'},
         'id': '4bf58dd8d48988d1f2941735',
         'name': 'Sporting Goods Shop',
         'pluralName': 'Sporting Goods Shops',
         'primary': True,
         'shortName': 'Sporting Goods'}],
       'id': '4b784457f964a520cdc02ee3',
       'location': {'address': '604 Washington St',
        'cc': 'US',
        'city': 'Hoboken',
        'country': 'United States',
        'crossStreet': '6th St',
        'distance': 280,
        'formattedAddress': ['604 Washington St (6th St)',
         'Hoboken, NJ 07030',
         'United States'],
        'labeledLatLngs': [{'label': 'display',
          'lat': 40.74364646969993,
          'lng': -74.02907764645921}],
        'lat': 40.74364646969993,
        'lng': -74.02907764645921,
        'postalCode': '07030',
        'state': 'NJ'},
       'name': 'Fleet Feet Sports',
       'photos': {'count': 0, 'groups': []}}},
     {'reasons': {'count': 0,
       'items': [{'reasonName': 'globalInteractionReason',
         'summary': 'This spot is popular',
         'type': 'general'}]},
      'referralId': 'e-0-4a7eff1cf964a5206ff21fe3-13',
      'venue': {'categories': [{'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/food/default_',
          'suffix': '.png'},
         'id': '4bf58dd8d48988d14e941735',
         'name': 'American Restaurant',
         'pluralName': 'American Restaurants',
         'primary': True,
         'shortName': 'American'}],
       'id': '4a7eff1cf964a5206ff21fe3',
       'location': {'address': '61 6th St',
        'cc': 'US',
        'city': 'Hoboken',
        'country': 'United States',
        'crossStreet': 'at Court St',
        'distance': 317,
        'formattedAddress': ['61 6th St (at Court St)',
         'Hoboken, NJ 07030',
         'United States'],
        'labeledLatLngs': [{'label': 'display',
          'lat': 40.74332213312085,
          'lng': -74.02861473442614}],
        'lat': 40.74332213312085,
        'lng': -74.02861473442614,
        'postalCode': '07030',
        'state': 'NJ'},
       'name': 'Court Street Bar & Restaurant',
       'photos': {'count': 0, 'groups': []},
       'venuePage': {'id': '84003162'}}},
     {'reasons': {'count': 0,
       'items': [{'reasonName': 'globalInteractionReason',
         'summary': 'This spot is popular',
         'type': 'general'}]},
      'referralId': 'e-0-51c892e02fc6ae7a5278a97e-14',
      'venue': {'categories': [{'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/building/gym_',
          'suffix': '.png'},
         'id': '4bf58dd8d48988d175941735',
         'name': 'Gym / Fitness Center',
         'pluralName': 'Gyms or Fitness Centers',
         'primary': True,
         'shortName': 'Gym / Fitness'}],
       'id': '51c892e02fc6ae7a5278a97e',
       'location': {'address': '333 Newark street',
        'cc': 'US',
        'city': 'Hoboken',
        'country': 'United States',
        'distance': 254,
        'formattedAddress': ['333 Newark street',
         'Hoboken, NJ 07030',
         'United States'],
        'labeledLatLngs': [{'label': 'display',
          'lat': 40.745562683353086,
          'lng': -74.0328410494832}],
        'lat': 40.745562683353086,
        'lng': -74.0328410494832,
        'postalCode': '07030',
        'state': 'NJ'},
       'name': 'Hudson River Athletics',
       'photos': {'count': 0, 'groups': []},
       'venuePage': {'id': '68113182'}}},
     {'reasons': {'count': 0,
       'items': [{'reasonName': 'globalInteractionReason',
         'summary': 'This spot is popular',
         'type': 'general'}]},
      'referralId': 'e-0-49f37b88f964a520a26a1fe3-15',
      'venue': {'categories': [{'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/food/coffeeshop_',
          'suffix': '.png'},
         'id': '4bf58dd8d48988d1e0931735',
         'name': 'Coffee Shop',
         'pluralName': 'Coffee Shops',
         'primary': True,
         'shortName': 'Coffee Shop'}],
       'id': '49f37b88f964a520a26a1fe3',
       'location': {'address': '338 Bloomfield St',
        'cc': 'US',
        'city': 'Hoboken',
        'country': 'United States',
        'crossStreet': 'at 4th St',
        'distance': 266,
        'formattedAddress': ['338 Bloomfield St (at 4th St)',
         'Hoboken, NJ 07030',
         'United States'],
        'labeledLatLngs': [{'label': 'display',
          'lat': 40.74137492076492,
          'lng': -74.0305153338276}],
        'lat': 40.74137492076492,
        'lng': -74.0305153338276,
        'postalCode': '07030',
        'state': 'NJ'},
       'name': 'Empire Coffee & Tea',
       'photos': {'count': 0, 'groups': []},
       'venuePage': {'id': '119534416'}}},
     {'reasons': {'count': 0,
       'items': [{'reasonName': 'globalInteractionReason',
         'summary': 'This spot is popular',
         'type': 'general'}]},
      'referralId': 'e-0-4d9368407b5ea1437d14c8b8-16',
      'venue': {'categories': [{'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/food/falafel_',
          'suffix': '.png'},
         'id': '4bf58dd8d48988d10b941735',
         'name': 'Falafel Restaurant',
         'pluralName': 'Falafel Restaurants',
         'primary': True,
         'shortName': 'Falafel'}],
       'delivery': {'id': '277402',
        'provider': {'icon': {'name': '/delivery_provider_grubhub_20180129.png',
          'prefix': 'https://fastly.4sqi.net/img/general/cap/',
          'sizes': [40, 50]},
         'name': 'grubhub'},
        'url': 'https://www.grubhub.com/restaurant/mamouns-falafel-restaurant-502-washington-st-hoboken/277402?affiliate=1131&utm_source=foursquare-affiliate-network&utm_medium=affiliate&utm_campaign=1131&utm_content=277402'},
       'id': '4d9368407b5ea1437d14c8b8',
       'location': {'address': '502 Washington St',
        'cc': 'US',
        'city': 'Hoboken',
        'country': 'United States',
        'crossStreet': 'at 5th St',
        'distance': 269,
        'formattedAddress': ['502 Washington St (at 5th St)',
         'Hoboken, NJ 07030',
         'United States'],
        'labeledLatLngs': [{'label': 'display',
          'lat': 40.742302660378776,
          'lng': -74.02946547322561}],
        'lat': 40.742302660378776,
        'lng': -74.02946547322561,
        'postalCode': '07030',
        'state': 'NJ'},
       'name': "Mamoun's Falafel",
       'photos': {'count': 0, 'groups': []},
       'venuePage': {'id': '43307012'}}},
     {'reasons': {'count': 0,
       'items': [{'reasonName': 'globalInteractionReason',
         'summary': 'This spot is popular',
         'type': 'general'}]},
      'referralId': 'e-0-49f26862f964a520296a1fe3-17',
      'venue': {'categories': [{'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/food/default_',
          'suffix': '.png'},
         'id': '4bf58dd8d48988d14e941735',
         'name': 'American Restaurant',
         'pluralName': 'American Restaurants',
         'primary': True,
         'shortName': 'American'}],
       'id': '49f26862f964a520296a1fe3',
       'location': {'address': '232 Willow Ave',
        'cc': 'US',
        'city': 'Hoboken',
        'country': 'United States',
        'crossStreet': 'at 3rd St',
        'distance': 321,
        'formattedAddress': ['232 Willow Ave (at 3rd St)',
         'Hoboken, NJ 07030',
         'United States'],
        'labeledLatLngs': [{'label': 'display',
          'lat': 40.74064008523523,
          'lng': -74.03382573472544}],
        'lat': 40.74064008523523,
        'lng': -74.03382573472544,
        'postalCode': '07030',
        'state': 'NJ'},
       'name': "Zack's Oak Bar & Restaurant",
       'photos': {'count': 0, 'groups': []},
       'venuePage': {'id': '60041199'}}},
     {'reasons': {'count': 0,
       'items': [{'reasonName': 'globalInteractionReason',
         'summary': 'This spot is popular',
         'type': 'general'}]},
      'referralId': 'e-0-4a7b5b6bf964a520c8ea1fe3-18',
      'venue': {'categories': [{'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/food/italian_',
          'suffix': '.png'},
         'id': '4bf58dd8d48988d110941735',
         'name': 'Italian Restaurant',
         'pluralName': 'Italian Restaurants',
         'primary': True,
         'shortName': 'Italian'}],
       'id': '4a7b5b6bf964a520c8ea1fe3',
       'location': {'address': '423 Bloomfield St',
        'cc': 'US',
        'city': 'Hoboken',
        'country': 'United States',
        'crossStreet': '5th St.',
        'distance': 214,
        'formattedAddress': ['423 Bloomfield St (5th St.)',
         'Hoboken, NJ 07030',
         'United States'],
        'labeledLatLngs': [{'label': 'display',
          'lat': 40.742278125391024,
          'lng': -74.03021807751502}],
        'lat': 40.742278125391024,
        'lng': -74.03021807751502,
        'postalCode': '07030',
        'state': 'NJ'},
       'name': 'Cafe Michelina',
       'photos': {'count': 0, 'groups': []}}},
     {'reasons': {'count': 0,
       'items': [{'reasonName': 'globalInteractionReason',
         'summary': 'This spot is popular',
         'type': 'general'}]},
      'referralId': 'e-0-4ca50f407334236a60ef1258-19',
      'venue': {'categories': [{'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/food/pizza_',
          'suffix': '.png'},
         'id': '4bf58dd8d48988d1ca941735',
         'name': 'Pizza Place',
         'pluralName': 'Pizza Places',
         'primary': True,
         'shortName': 'Pizza'}],
       'id': '4ca50f407334236a60ef1258',
       'location': {'address': '411 Washington St',
        'cc': 'US',
        'city': 'Hoboken',
        'country': 'United States',
        'crossStreet': 'btwn 4th and 5th',
        'distance': 297,
        'formattedAddress': ['411 Washington St (btwn 4th and 5th)',
         'Hoboken, NJ 07030',
         'United States'],
        'labeledLatLngs': [{'label': 'display',
          'lat': 40.74167406908621,
          'lng': -74.02957750973907}],
        'lat': 40.74167406908621,
        'lng': -74.02957750973907,
        'postalCode': '07030',
        'state': 'NJ'},
       'name': "Grimaldi's",
       'photos': {'count': 0, 'groups': []},
       'venuePage': {'id': '57687383'}}},
     {'reasons': {'count': 0,
       'items': [{'reasonName': 'globalInteractionReason',
         'summary': 'This spot is popular',
         'type': 'general'}]},
      'referralId': 'e-0-4cdb36c1958f236a15a7ab03-20',
      'venue': {'categories': [{'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/food/sushi_',
          'suffix': '.png'},
         'id': '4bf58dd8d48988d1d2941735',
         'name': 'Sushi Restaurant',
         'pluralName': 'Sushi Restaurants',
         'primary': True,
         'shortName': 'Sushi'}],
       'delivery': {'id': '70931',
        'provider': {'icon': {'name': '/delivery_provider_grubhub_20180129.png',
          'prefix': 'https://fastly.4sqi.net/img/general/cap/',
          'sizes': [40, 50]},
         'name': 'grubhub'},
        'url': 'https://www.grubhub.com/restaurant/robongi-restaurant-hoboken-520-washington-st-hoboken/70931?affiliate=1131&utm_source=foursquare-affiliate-network&utm_medium=affiliate&utm_campaign=1131&utm_content=70931'},
       'id': '4cdb36c1958f236a15a7ab03',
       'location': {'address': '520 Washington St',
        'cc': 'US',
        'city': 'Hoboken',
        'country': 'United States',
        'crossStreet': '5th St.',
        'distance': 265,
        'formattedAddress': ['520 Washington St (5th St.)',
         'Hoboken, NJ 07030',
         'United States'],
        'labeledLatLngs': [{'label': 'display',
          'lat': 40.74287866510769,
          'lng': -74.0292797646991}],
        'lat': 40.74287866510769,
        'lng': -74.0292797646991,
        'postalCode': '07030',
        'state': 'NJ'},
       'name': 'Robongi',
       'photos': {'count': 0, 'groups': []}}},
     {'reasons': {'count': 0,
       'items': [{'reasonName': 'globalInteractionReason',
         'summary': 'This spot is popular',
         'type': 'general'}]},
      'referralId': 'e-0-4d3b7bdd687ca35d1e8a94c4-21',
      'venue': {'categories': [{'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/food/bakery_',
          'suffix': '.png'},
         'id': '4bf58dd8d48988d16a941735',
         'name': 'Bakery',
         'pluralName': 'Bakeries',
         'primary': True,
         'shortName': 'Bakery'}],
       'id': '4d3b7bdd687ca35d1e8a94c4',
       'location': {'address': '332 Washington St',
        'cc': 'US',
        'city': 'Hoboken',
        'country': 'United States',
        'crossStreet': 'at 4th St',
        'distance': 323,
        'formattedAddress': ['332 Washington St (at 4th St)',
         'Hoboken, NJ 07030',
         'United States'],
        'labeledLatLngs': [{'label': 'display',
          'lat': 40.74114653903486,
          'lng': -74.02980981300425}],
        'lat': 40.74114653903486,
        'lng': -74.02980981300425,
        'postalCode': '07030',
        'state': 'NJ'},
       'name': 'Old German Bakery',
       'photos': {'count': 0, 'groups': []}}},
     {'reasons': {'count': 0,
       'items': [{'reasonName': 'globalInteractionReason',
         'summary': 'This spot is popular',
         'type': 'general'}]},
      'referralId': 'e-0-53ed3b37498e4151087521a9-22',
      'venue': {'categories': [{'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/food/burger_',
          'suffix': '.png'},
         'id': '4bf58dd8d48988d16c941735',
         'name': 'Burger Joint',
         'pluralName': 'Burger Joints',
         'primary': True,
         'shortName': 'Burgers'}],
       'delivery': {'id': '297961',
        'provider': {'icon': {'name': '/delivery_provider_grubhub_20180129.png',
          'prefix': 'https://fastly.4sqi.net/img/general/cap/',
          'sizes': [40, 50]},
         'name': 'grubhub'},
        'url': 'https://www.grubhub.com/restaurant/bareburger-515-washington-st-hoboken/297961?affiliate=1131&utm_source=foursquare-affiliate-network&utm_medium=affiliate&utm_campaign=1131&utm_content=297961'},
       'id': '53ed3b37498e4151087521a9',
       'location': {'address': '515 Washington St',
        'cc': 'US',
        'city': 'Hoboken',
        'country': 'United States',
        'distance': 286,
        'formattedAddress': ['515 Washington St',
         'Hoboken, NJ 07030',
         'United States'],
        'labeledLatLngs': [{'label': 'display',
          'lat': 40.742693954701245,
          'lng': -74.02907006819072}],
        'lat': 40.742693954701245,
        'lng': -74.02907006819072,
        'postalCode': '07030',
        'state': 'NJ'},
       'name': 'Bareburger',
       'photos': {'count': 0, 'groups': []}}},
     {'reasons': {'count': 0,
       'items': [{'reasonName': 'globalInteractionReason',
         'summary': 'This spot is popular',
         'type': 'general'}]},
      'referralId': 'e-0-49e2a407f964a52045621fe3-23',
      'venue': {'categories': [{'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/food/argentinian_',
          'suffix': '.png'},
         'id': '4bf58dd8d48988d1cd941735',
         'name': 'South American Restaurant',
         'pluralName': 'South American Restaurants',
         'primary': True,
         'shortName': 'South American'}],
       'id': '49e2a407f964a52045621fe3',
       'location': {'address': '233 Clinton St',
        'cc': 'US',
        'city': 'Hoboken',
        'country': 'United States',
        'crossStreet': 'btwn 2nd & 3rd St.',
        'distance': 334,
        'formattedAddress': ['233 Clinton St (btwn 2nd & 3rd St.)',
         'Hoboken, NJ 07030',
         'United States'],
        'labeledLatLngs': [{'label': 'display',
          'lat': 40.7408073760094,
          'lng': -74.03458175351923}],
        'lat': 40.7408073760094,
        'lng': -74.03458175351923,
        'postalCode': '07030',
        'state': 'NJ'},
       'name': 'Cucharamama',
       'photos': {'count': 0, 'groups': []}}},
     {'reasons': {'count': 0,
       'items': [{'reasonName': 'globalInteractionReason',
         'summary': 'This spot is popular',
         'type': 'general'}]},
      'referralId': 'e-0-4d4218cd607b6dcb31df08c6-24',
      'venue': {'categories': [{'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/nightlife/pub_',
          'suffix': '.png'},
         'id': '4bf58dd8d48988d11b941735',
         'name': 'Pub',
         'pluralName': 'Pubs',
         'primary': True,
         'shortName': 'Pub'}],
       'id': '4d4218cd607b6dcb31df08c6',
       'location': {'address': '239 Bloomfield St',
        'cc': 'US',
        'city': 'Hoboken',
        'country': 'United States',
        'crossStreet': 'at 3rd St',
        'distance': 378,
        'formattedAddress': ['239 Bloomfield St (at 3rd St)',
         'Hoboken, NJ 07030',
         'United States'],
        'labeledLatLngs': [{'label': 'display',
          'lat': 40.74010486484954,
          'lng': -74.03086819003738}],
        'lat': 40.74010486484954,
        'lng': -74.03086819003738,
        'postalCode': '07030',
        'state': 'NJ'},
       'name': 'Cork City Pub',
       'photos': {'count': 0, 'groups': []},
       'venuePage': {'id': '50649981'}}},
     {'reasons': {'count': 0,
       'items': [{'reasonName': 'globalInteractionReason',
         'summary': 'This spot is popular',
         'type': 'general'}]},
      'referralId': 'e-0-49dfb562f964a52001611fe3-25',
      'venue': {'categories': [{'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/food/icecream_',
          'suffix': '.png'},
         'id': '4bf58dd8d48988d1c9941735',
         'name': 'Ice Cream Shop',
         'pluralName': 'Ice Cream Shops',
         'primary': True,
         'shortName': 'Ice Cream'}],
       'delivery': {'id': '78617',
        'provider': {'icon': {'name': '/delivery_provider_grubhub_20180129.png',
          'prefix': 'https://fastly.4sqi.net/img/general/cap/',
          'sizes': [40, 50]},
         'name': 'grubhub'},
        'url': 'https://www.grubhub.com/restaurant/ben--jerrys-405-washington-st-hoboken/78617?affiliate=1131&utm_source=foursquare-affiliate-network&utm_medium=affiliate&utm_campaign=1131&utm_content=78617'},
       'id': '49dfb562f964a52001611fe3',
       'location': {'address': '405 Washington St',
        'cc': 'US',
        'city': 'Hoboken',
        'country': 'United States',
        'distance': 321,
        'formattedAddress': ['405 Washington St',
         'Hoboken, NJ 07030',
         'United States'],
        'labeledLatLngs': [{'label': 'display',
          'lat': 40.7414305,
          'lng': -74.0294837}],
        'lat': 40.7414305,
        'lng': -74.0294837,
        'postalCode': '07030',
        'state': 'NJ'},
       'name': "Ben & Jerry's",
       'photos': {'count': 0, 'groups': []}}},
     {'reasons': {'count': 0,
       'items': [{'reasonName': 'globalInteractionReason',
         'summary': 'This spot is popular',
         'type': 'general'}]},
      'referralId': 'e-0-4b93cf35f964a520c65234e3-26',
      'venue': {'categories': [{'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/parks_outdoors/dogrun_',
          'suffix': '.png'},
         'id': '4bf58dd8d48988d1e5941735',
         'name': 'Dog Run',
         'pluralName': 'Dog Runs',
         'primary': True,
         'shortName': 'Dog Run'}],
       'id': '4b93cf35f964a520c65234e3',
       'location': {'address': '401 Willow Ave.',
        'cc': 'US',
        'city': 'Hoboken',
        'country': 'United States',
        'crossStreet': '4th & Willow',
        'distance': 134,
        'formattedAddress': ['401 Willow Ave. (4th & Willow)',
         'Hoboken, NJ 07030',
         'United States'],
        'labeledLatLngs': [{'label': 'display',
          'lat': 40.742203138352124,
          'lng': -74.03303786884511}],
        'lat': 40.742203138352124,
        'lng': -74.03303786884511,
        'postalCode': '07030',
        'state': 'NJ'},
       'name': 'Church Square Dog Park',
       'photos': {'count': 0, 'groups': []}}},
     {'reasons': {'count': 0,
       'items': [{'reasonName': 'globalInteractionReason',
         'summary': 'This spot is popular',
         'type': 'general'}]},
      'referralId': 'e-0-4eb1b6859adfb95b77765bf9-27',
      'venue': {'categories': [{'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/food/cuban_',
          'suffix': '.png'},
         'id': '4bf58dd8d48988d154941735',
         'name': 'Cuban Restaurant',
         'pluralName': 'Cuban Restaurants',
         'primary': True,
         'shortName': 'Cuban'}],
       'id': '4eb1b6859adfb95b77765bf9',
       'location': {'address': '333 Washington St',
        'cc': 'US',
        'city': 'Hoboken',
        'country': 'United States',
        'crossStreet': '4th street',
        'distance': 338,
        'formattedAddress': ['333 Washington St (4th street)',
         'Hoboken, NJ 07030',
         'United States'],
        'labeledLatLngs': [{'label': 'display',
          'lat': 40.74101245470124,
          'lng': -74.02973234956126}],
        'lat': 40.74101245470124,
        'lng': -74.02973234956126,
        'postalCode': '07030',
        'state': 'NJ'},
       'name': 'The Cuban',
       'photos': {'count': 0, 'groups': []}}},
     {'reasons': {'count': 0,
       'items': [{'reasonName': 'globalInteractionReason',
         'summary': 'This spot is popular',
         'type': 'general'}]},
      'referralId': 'e-0-57127a9d498e648da026a585-28',
      'venue': {'categories': [{'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/shops/gamingcafe_',
          'suffix': '.png'},
         'id': '4bf58dd8d48988d18d941735',
         'name': 'Gaming Cafe',
         'pluralName': 'Gaming Cafes',
         'primary': True,
         'shortName': 'Gaming Cafe'}],
       'id': '57127a9d498e648da026a585',
       'location': {'address': '519 Washington St',
        'cc': 'US',
        'city': 'Hoboken',
        'country': 'United States',
        'crossStreet': 'btwn 5th & 6th',
        'distance': 274,
        'formattedAddress': ['519 Washington St (btwn 5th & 6th)',
         'Hoboken, NJ 07030',
         'United States'],
        'labeledLatLngs': [{'label': 'display',
          'lat': 40.742812745885644,
          'lng': -74.02918367562344}],
        'lat': 40.742812745885644,
        'lng': -74.02918367562344,
        'postalCode': '07030',
        'state': 'NJ'},
       'name': 'Aether Game Cafe',
       'photos': {'count': 0, 'groups': []}}},
     {'reasons': {'count': 0,
       'items': [{'reasonName': 'globalInteractionReason',
         'summary': 'This spot is popular',
         'type': 'general'}]},
      'referralId': 'e-0-4a9ac1b1f964a520813220e3-29',
      'venue': {'categories': [{'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/food/pizza_',
          'suffix': '.png'},
         'id': '4bf58dd8d48988d1ca941735',
         'name': 'Pizza Place',
         'pluralName': 'Pizza Places',
         'primary': True,
         'shortName': 'Pizza'}],
       'id': '4a9ac1b1f964a520813220e3',
       'location': {'address': '622 Washington St',
        'cc': 'US',
        'city': 'Hoboken',
        'country': 'United States',
        'crossStreet': 'btwn 6th & 7th',
        'distance': 309,
        'formattedAddress': ['622 Washington St (btwn 6th & 7th)',
         'Hoboken, NJ 07030',
         'United States'],
        'labeledLatLngs': [{'label': 'display',
          'lat': 40.74419827547744,
          'lng': -74.02889639843647}],
        'lat': 40.74419827547744,
        'lng': -74.02889639843647,
        'postalCode': '07030',
        'state': 'NJ'},
       'name': "Benny Tudino's",
       'photos': {'count': 0, 'groups': []}}},
     {'reasons': {'count': 0,
       'items': [{'reasonName': 'globalInteractionReason',
         'summary': 'This spot is popular',
         'type': 'general'}]},
      'referralId': 'e-0-5bede1881953f3002c8bd30e-30',
      'venue': {'categories': [{'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/building/medical_opticalshop_',
          'suffix': '.png'},
         'id': '4d954afda243a5684865b473',
         'name': 'Optical Shop',
         'pluralName': 'Optical Shops',
         'primary': True,
         'shortName': 'Optical'}],
       'id': '5bede1881953f3002c8bd30e',
       'location': {'address': '538 Washington Street',
        'cc': 'US',
        'city': 'Hoboken',
        'country': 'United States',
        'distance': 250,
        'formattedAddress': ['538 Washington Street',
         'Hoboken, NJ 07030',
         'United States'],
        'labeledLatLngs': [{'label': 'display',
          'lat': 40.743458,
          'lng': -74.029408}],
        'lat': 40.743458,
        'lng': -74.029408,
        'postalCode': '07030',
        'state': 'NJ'},
       'name': 'Warby Parker',
       'photos': {'count': 0, 'groups': []}}},
     {'reasons': {'count': 0,
       'items': [{'reasonName': 'globalInteractionReason',
         'summary': 'This spot is popular',
         'type': 'general'}]},
      'referralId': 'e-0-4b5cd37cf964a520f04529e3-31',
      'venue': {'categories': [{'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/shops/gym_yogastudio_',
          'suffix': '.png'},
         'id': '4bf58dd8d48988d102941735',
         'name': 'Yoga Studio',
         'pluralName': 'Yoga Studios',
         'primary': True,
         'shortName': 'Yoga Studio'}],
       'id': '4b5cd37cf964a520f04529e3',
       'location': {'address': '618 Washington St',
        'cc': 'US',
        'city': 'Hoboken',
        'country': 'United States',
        'crossStreet': '6th St.',
        'distance': 297,
        'formattedAddress': ['618 Washington St (6th St.)',
         'Hoboken, NJ 07030',
         'United States'],
        'labeledLatLngs': [{'label': 'display',
          'lat': 40.744101667075846,
          'lng': -74.02900706503569}],
        'lat': 40.744101667075846,
        'lng': -74.02900706503569,
        'postalCode': '07030',
        'state': 'NJ'},
       'name': 'Surya Yoga Academy',
       'photos': {'count': 0, 'groups': []},
       'venuePage': {'id': '41891030'}}},
     {'reasons': {'count': 0,
       'items': [{'reasonName': 'globalInteractionReason',
         'summary': 'This spot is popular',
         'type': 'general'}]},
      'referralId': 'e-0-4eb039c4e5fa708105c903ac-32',
      'venue': {'categories': [{'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/shops/pet_store_',
          'suffix': '.png'},
         'id': '4bf58dd8d48988d100951735',
         'name': 'Pet Store',
         'pluralName': 'Pet Stores',
         'primary': True,
         'shortName': 'Pet Store'}],
       'id': '4eb039c4e5fa708105c903ac',
       'location': {'address': '524 Washington St',
        'cc': 'US',
        'city': 'Hoboken',
        'country': 'United States',
        'distance': 265,
        'formattedAddress': ['524 Washington St',
         'Hoboken, NJ 07030',
         'United States'],
        'labeledLatLngs': [{'label': 'display',
          'lat': 40.74303498433573,
          'lng': -74.02924404967993}],
        'lat': 40.74303498433573,
        'lng': -74.02924404967993,
        'postalCode': '07030',
        'state': 'NJ'},
       'name': 'Hoboken Pet',
       'photos': {'count': 0, 'groups': []},
       'venuePage': {'id': '37704701'}}},
     {'reasons': {'count': 0,
       'items': [{'reasonName': 'globalInteractionReason',
         'summary': 'This spot is popular',
         'type': 'general'}]},
      'referralId': 'e-0-4ad12c5ef964a5203ddd20e3-33',
      'venue': {'categories': [{'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/nightlife/pub_',
          'suffix': '.png'},
         'id': '4bf58dd8d48988d116941735',
         'name': 'Bar',
         'pluralName': 'Bars',
         'primary': True,
         'shortName': 'Bar'}],
       'id': '4ad12c5ef964a5203ddd20e3',
       'location': {'address': '501 Garden St',
        'cc': 'US',
        'city': 'Hoboken',
        'country': 'United States',
        'crossStreet': '5th Street',
        'distance': 130,
        'formattedAddress': ['501 Garden St (5th Street)',
         'Hoboken, NJ 07030',
         'United States'],
        'labeledLatLngs': [{'label': 'display',
          'lat': 40.742552761953135,
          'lng': -74.03119210059499}],
        'lat': 40.742552761953135,
        'lng': -74.03119210059499,
        'postalCode': '07030',
        'state': 'NJ'},
       'name': "Moran's Pub",
       'photos': {'count': 0, 'groups': []},
       'venuePage': {'id': '81088402'}}},
     {'reasons': {'count': 0,
       'items': [{'reasonName': 'globalInteractionReason',
         'summary': 'This spot is popular',
         'type': 'general'}]},
      'referralId': 'e-0-4a453f9cf964a520f4a71fe3-34',
      'venue': {'categories': [{'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/food/cuban_',
          'suffix': '.png'},
         'id': '4bf58dd8d48988d154941735',
         'name': 'Cuban Restaurant',
         'pluralName': 'Cuban Restaurants',
         'primary': True,
         'shortName': 'Cuban'}],
       'delivery': {'id': '622030',
        'provider': {'icon': {'name': '/delivery_provider_grubhub_20180129.png',
          'prefix': 'https://fastly.4sqi.net/img/general/cap/',
          'sizes': [40, 50]},
         'name': 'grubhub'},
        'url': 'https://www.grubhub.com/restaurant/zafra-301-willow-ave-hoboken/622030?affiliate=1131&utm_source=foursquare-affiliate-network&utm_medium=affiliate&utm_campaign=1131&utm_content=622030'},
       'id': '4a453f9cf964a520f4a71fe3',
       'location': {'address': '301 Willow Ave',
        'cc': 'US',
        'city': 'Hoboken',
        'country': 'United States',
        'crossStreet': 'at 3rd St',
        'distance': 318,
        'formattedAddress': ['301 Willow Ave (at 3rd St)',
         'Hoboken, NJ 07030',
         'United States'],
        'labeledLatLngs': [{'label': 'display',
          'lat': 40.740659471339825,
          'lng': -74.03380200797542}],
        'lat': 40.740659471339825,
        'lng': -74.03380200797542,
        'postalCode': '07030',
        'state': 'NJ'},
       'name': 'Zafra',
       'photos': {'count': 0, 'groups': []}}},
     {'reasons': {'count': 0,
       'items': [{'reasonName': 'globalInteractionReason',
         'summary': 'This spot is popular',
         'type': 'general'}]},
      'referralId': 'e-0-57f83f7acd10164c2ec1956f-35',
      'venue': {'categories': [{'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/food/seafood_',
          'suffix': '.png'},
         'id': '4bf58dd8d48988d1ce941735',
         'name': 'Seafood Restaurant',
         'pluralName': 'Seafood Restaurants',
         'primary': True,
         'shortName': 'Seafood'}],
       'id': '57f83f7acd10164c2ec1956f',
       'location': {'address': '155 3rd St',
        'cc': 'US',
        'city': 'Hoboken',
        'country': 'United States',
        'crossStreet': 'Bloomfield',
        'distance': 361,
        'formattedAddress': ['155 3rd St (Bloomfield)',
         'Hoboken, NJ 07030',
         'United States'],
        'labeledLatLngs': [{'label': 'display',
          'lat': 40.7401625051114,
          'lng': -74.03128399753021}],
        'lat': 40.7401625051114,
        'lng': -74.03128399753021,
        'postalCode': '07030',
        'state': 'NJ'},
       'name': 'Tutta Pesca',
       'photos': {'count': 0, 'groups': []}}},
     {'reasons': {'count': 0,
       'items': [{'reasonName': 'globalInteractionReason',
         'summary': 'This spot is popular',
         'type': 'general'}]},
      'referralId': 'e-0-4c03f2fe39d476b0f5c530a7-36',
      'venue': {'categories': [{'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/food/latinamerican_',
          'suffix': '.png'},
         'id': '4bf58dd8d48988d1be941735',
         'name': 'Latin American Restaurant',
         'pluralName': 'Latin American Restaurants',
         'primary': True,
         'shortName': 'Latin American'}],
       'id': '4c03f2fe39d476b0f5c530a7',
       'location': {'address': '260 3rd St',
        'cc': 'US',
        'city': 'Hoboken',
        'country': 'United States',
        'crossStreet': 'Willow Ave.',
        'distance': 313,
        'formattedAddress': ['260 3rd St (Willow Ave.)',
         'Hoboken, NJ 07030',
         'United States'],
        'labeledLatLngs': [{'label': 'display',
          'lat': 40.74061999725131,
          'lng': -74.03349023134471}],
        'lat': 40.74061999725131,
        'lng': -74.03349023134471,
        'postalCode': '07030',
        'state': 'NJ'},
       'name': 'Ultramarinos',
       'photos': {'count': 0, 'groups': []}}},
     {'reasons': {'count': 0,
       'items': [{'reasonName': 'globalInteractionReason',
         'summary': 'This spot is popular',
         'type': 'general'}]},
      'referralId': 'e-0-4c60c4a1de6920a111ed9664-37',
      'venue': {'categories': [{'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/food/pizza_',
          'suffix': '.png'},
         'id': '4bf58dd8d48988d1ca941735',
         'name': 'Pizza Place',
         'pluralName': 'Pizza Places',
         'primary': True,
         'shortName': 'Pizza'}],
       'delivery': {'id': '347175',
        'provider': {'icon': {'name': '/delivery_provider_grubhub_20180129.png',
          'prefix': 'https://fastly.4sqi.net/img/general/cap/',
          'sizes': [40, 50]},
         'name': 'grubhub'},
        'url': 'https://www.grubhub.com/restaurant/dozzino-534-adams-st-hoboken/347175?affiliate=1131&utm_source=foursquare-affiliate-network&utm_medium=affiliate&utm_campaign=1131&utm_content=347175'},
       'id': '4c60c4a1de6920a111ed9664',
       'location': {'address': '534 Adams St',
        'cc': 'US',
        'city': 'Hoboken',
        'country': 'United States',
        'crossStreet': 'Sixth',
        'distance': 310,
        'formattedAddress': ['534 Adams St (Sixth)',
         'Hoboken, NJ 07030',
         'United States'],
        'labeledLatLngs': [{'label': 'display',
          'lat': 40.74461204537549,
          'lng': -74.0356316258318}],
        'lat': 40.74461204537549,
        'lng': -74.0356316258318,
        'postalCode': '07030',
        'state': 'NJ'},
       'name': 'Dozzino',
       'photos': {'count': 0, 'groups': []}}},
     {'reasons': {'count': 0,
       'items': [{'reasonName': 'globalInteractionReason',
         'summary': 'This spot is popular',
         'type': 'general'}]},
      'referralId': 'e-0-527f3d1711d2f7f001c656b2-38',
      'venue': {'categories': [{'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/food/italian_',
          'suffix': '.png'},
         'id': '4bf58dd8d48988d110941735',
         'name': 'Italian Restaurant',
         'pluralName': 'Italian Restaurants',
         'primary': True,
         'shortName': 'Italian'}],
       'delivery': {'id': '301123',
        'provider': {'icon': {'name': '/delivery_provider_grubhub_20180129.png',
          'prefix': 'https://fastly.4sqi.net/img/general/cap/',
          'sizes': [40, 50]},
         'name': 'grubhub'},
        'url': 'https://www.grubhub.com/restaurant/otto-strada-743-park-aveune-hoboken/301123?affiliate=1131&utm_source=foursquare-affiliate-network&utm_medium=affiliate&utm_campaign=1131&utm_content=301123'},
       'id': '527f3d1711d2f7f001c656b2',
       'location': {'address': '743 Park Ave',
        'cc': 'US',
        'city': 'Hoboken',
        'country': 'United States',
        'crossStreet': '8th Street',
        'distance': 381,
        'formattedAddress': ['743 Park Ave (8th Street)',
         'Hoboken, NJ 07030',
         'United States'],
        'labeledLatLngs': [{'label': 'display',
          'lat': 40.74660389157448,
          'lng': -74.03116055588913}],
        'lat': 40.74660389157448,
        'lng': -74.03116055588913,
        'postalCode': '07030',
        'state': 'NJ'},
       'name': 'Otto Strada',
       'photos': {'count': 0, 'groups': []}}},
     {'reasons': {'count': 0,
       'items': [{'reasonName': 'globalInteractionReason',
         'summary': 'This spot is popular',
         'type': 'general'}]},
      'referralId': 'e-0-4a9578dff964a520562320e3-39',
      'venue': {'categories': [{'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/food/japanese_',
          'suffix': '.png'},
         'id': '4bf58dd8d48988d111941735',
         'name': 'Japanese Restaurant',
         'pluralName': 'Japanese Restaurants',
         'primary': True,
         'shortName': 'Japanese'}],
       'delivery': {'id': '91087',
        'provider': {'icon': {'name': '/delivery_provider_grubhub_20180129.png',
          'prefix': 'https://fastly.4sqi.net/img/general/cap/',
          'sizes': [40, 50]},
         'name': 'grubhub'},
        'url': 'https://www.grubhub.com/restaurant/illuzion-337-washington-st-hoboken/91087?affiliate=1131&utm_source=foursquare-affiliate-network&utm_medium=affiliate&utm_campaign=1131&utm_content=91087'},
       'id': '4a9578dff964a520562320e3',
       'location': {'address': '337 Washington St',
        'cc': 'US',
        'city': 'Hoboken',
        'country': 'United States',
        'crossStreet': 'at Fourth St',
        'distance': 330,
        'formattedAddress': ['337 Washington St (at Fourth St)',
         'Hoboken, NJ 07030',
         'United States'],
        'labeledLatLngs': [{'label': 'display',
          'lat': 40.74122926622237,
          'lng': -74.02958214540159}],
        'lat': 40.74122926622237,
        'lng': -74.02958214540159,
        'postalCode': '07030',
        'state': 'NJ'},
       'name': 'Illuzion',
       'photos': {'count': 0, 'groups': []}}},
     {'reasons': {'count': 0,
       'items': [{'reasonName': 'globalInteractionReason',
         'summary': 'This spot is popular',
         'type': 'general'}]},
      'referralId': 'e-0-58e7ed715f67173549fe6246-40',
      'venue': {'categories': [{'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/food/coffeeshop_',
          'suffix': '.png'},
         'id': '4bf58dd8d48988d1e0931735',
         'name': 'Coffee Shop',
         'pluralName': 'Coffee Shops',
         'primary': True,
         'shortName': 'Coffee Shop'}],
       'id': '58e7ed715f67173549fe6246',
       'location': {'address': '409 Washington St',
        'cc': 'US',
        'city': 'Hoboken',
        'country': 'United States',
        'distance': 314,
        'formattedAddress': ['409 Washington St',
         'Hoboken, NJ 07030',
         'United States'],
        'labeledLatLngs': [{'label': 'display',
          'lat': 40.7416010355079,
          'lng': -74.02940965731437}],
        'lat': 40.7416010355079,
        'lng': -74.02940965731437,
        'postalCode': '07030',
        'state': 'NJ'},
       'name': 'Bluestone Lane',
       'photos': {'count': 0, 'groups': []}}},
     {'reasons': {'count': 0,
       'items': [{'reasonName': 'globalInteractionReason',
         'summary': 'This spot is popular',
         'type': 'general'}]},
      'referralId': 'e-0-4b7415e2f964a52023c72de3-41',
      'venue': {'categories': [{'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/shops/apparel_shoestore_',
          'suffix': '.png'},
         'id': '52f2ab2ebcbc57f1066b8b39',
         'name': 'Shoe Repair',
         'pluralName': 'Shoe Repair Shops',
         'primary': True,
         'shortName': 'Shoe Repair'}],
       'id': '4b7415e2f964a52023c72de3',
       'location': {'address': '266 7th St',
        'cc': 'US',
        'city': 'Hoboken',
        'country': 'United States',
        'crossStreet': 'at Willow Ave',
        'distance': 231,
        'formattedAddress': ['266 7th St (at Willow Ave)',
         'Hoboken, NJ 07030',
         'United States'],
        'labeledLatLngs': [{'label': 'display',
          'lat': 40.7453816269899,
          'lng': -74.03222690401293}],
        'lat': 40.7453816269899,
        'lng': -74.03222690401293,
        'postalCode': '07030',
        'state': 'NJ'},
       'name': "Giovanni D'Italia Shoe Repair",
       'photos': {'count': 0, 'groups': []}}},
     {'reasons': {'count': 0,
       'items': [{'reasonName': 'globalInteractionReason',
         'summary': 'This spot is popular',
         'type': 'general'}]},
      'referralId': 'e-0-49ee57f6f964a5204f681fe3-42',
      'venue': {'categories': [{'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/food/mexican_',
          'suffix': '.png'},
         'id': '4bf58dd8d48988d1c1941735',
         'name': 'Mexican Restaurant',
         'pluralName': 'Mexican Restaurants',
         'primary': True,
         'shortName': 'Mexican'}],
       'delivery': {'id': '263911',
        'provider': {'icon': {'name': '/delivery_provider_grubhub_20180129.png',
          'prefix': 'https://fastly.4sqi.net/img/general/cap/',
          'sizes': [40, 50]},
         'name': 'grubhub'},
        'url': 'https://www.grubhub.com/restaurant/hoboken-burrito-209-4th-st-hoboken/263911?affiliate=1131&utm_source=foursquare-affiliate-network&utm_medium=affiliate&utm_campaign=1131&utm_content=263911'},
       'id': '49ee57f6f964a5204f681fe3',
       'location': {'address': '209 4th St',
        'cc': 'US',
        'city': 'Hoboken',
        'country': 'United States',
        'crossStreet': 'Park & Garden',
        'distance': 170,
        'formattedAddress': ['209 4th St (Park & Garden)',
         'Hoboken, NJ 07030',
         'United States'],
        'labeledLatLngs': [{'label': 'display',
          'lat': 40.74182226201247,
          'lng': -74.03188093618373}],
        'lat': 40.74182226201247,
        'lng': -74.03188093618373,
        'postalCode': '07030',
        'state': 'NJ'},
       'name': 'Hoboken Burrito',
       'photos': {'count': 0, 'groups': []},
       'venuePage': {'id': '84840218'}}},
     {'reasons': {'count': 0,
       'items': [{'reasonName': 'globalInteractionReason',
         'summary': 'This spot is popular',
         'type': 'general'}]},
      'referralId': 'e-0-4bddbf6be75c0f47f171c503-43',
      'venue': {'categories': [{'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/shops/apparel_boutique_',
          'suffix': '.png'},
         'id': '4bf58dd8d48988d104951735',
         'name': 'Boutique',
         'pluralName': 'Boutiques',
         'primary': True,
         'shortName': 'Boutique'}],
       'id': '4bddbf6be75c0f47f171c503',
       'location': {'address': '620 Washington St',
        'cc': 'US',
        'city': 'Hoboken',
        'country': 'United States',
        'crossStreet': 'Between 6th and 7th',
        'distance': 293,
        'formattedAddress': ['620 Washington St (Between 6th and 7th)',
         'Hoboken, NJ 07030',
         'United States'],
        'labeledLatLngs': [{'label': 'display',
          'lat': 40.744128054776574,
          'lng': -74.02907041180875}],
        'lat': 40.744128054776574,
        'lng': -74.02907041180875,
        'postalCode': '07030',
        'state': 'NJ'},
       'name': 'Townhouse No 620',
       'photos': {'count': 0, 'groups': []}}},
     {'reasons': {'count': 0,
       'items': [{'reasonName': 'globalInteractionReason',
         'summary': 'This spot is popular',
         'type': 'general'}]},
      'referralId': 'e-0-5a888bc2c47cf91545daec08-44',
      'venue': {'categories': [{'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/food/coffeeshop_',
          'suffix': '.png'},
         'id': '4bf58dd8d48988d1e0931735',
         'name': 'Coffee Shop',
         'pluralName': 'Coffee Shops',
         'primary': True,
         'shortName': 'Coffee Shop'}],
       'id': '5a888bc2c47cf91545daec08',
       'location': {'address': '534 Washington St',
        'cc': 'US',
        'city': 'Hoboken',
        'country': 'United States',
        'distance': 273,
        'formattedAddress': ['534 Washington St',
         'Hoboken, NJ 07030',
         'United States'],
        'labeledLatLngs': [{'label': 'display',
          'lat': 40.743367915985544,
          'lng': -74.02913501175979}],
        'lat': 40.743367915985544,
        'lng': -74.02913501175979,
        'postalCode': '07030',
        'state': 'NJ'},
       'name': 'Jefferson’s Coffee',
       'photos': {'count': 0, 'groups': []}}},
     {'reasons': {'count': 0,
       'items': [{'reasonName': 'globalInteractionReason',
         'summary': 'This spot is popular',
         'type': 'general'}]},
      'referralId': 'e-0-4a8da189f964a520501020e3-45',
      'venue': {'categories': [{'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/food/deli_',
          'suffix': '.png'},
         'id': '4bf58dd8d48988d1c5941735',
         'name': 'Sandwich Place',
         'pluralName': 'Sandwich Places',
         'primary': True,
         'shortName': 'Sandwiches'}],
       'delivery': {'id': '1024672',
        'provider': {'icon': {'name': '/delivery_provider_grubhub_20180129.png',
          'prefix': 'https://fastly.4sqi.net/img/general/cap/',
          'sizes': [40, 50]},
         'name': 'grubhub'},
        'url': 'https://www.grubhub.com/restaurant/vitos-italian-deli-806-washington-st-hoboken/1024672?affiliate=1131&utm_source=foursquare-affiliate-network&utm_medium=affiliate&utm_campaign=1131&utm_content=1024672'},
       'id': '4a8da189f964a520501020e3',
       'location': {'address': '806 Washington St',
        'cc': 'US',
        'city': 'Hoboken',
        'country': 'United States',
        'crossStreet': '8th',
        'distance': 486,
        'formattedAddress': ['806 Washington St (8th)',
         'Hoboken, NJ 07030',
         'United States'],
        'labeledLatLngs': [{'label': 'display',
          'lat': 40.746401,
          'lng': -74.02831}],
        'lat': 40.746401,
        'lng': -74.02831,
        'postalCode': '07030',
        'state': 'NJ'},
       'name': "Vito's Italian Deli",
       'photos': {'count': 0, 'groups': []},
       'venuePage': {'id': '94363093'}}},
     {'reasons': {'count': 0,
       'items': [{'reasonName': 'globalInteractionReason',
         'summary': 'This spot is popular',
         'type': 'general'}]},
      'referralId': 'e-0-58c470fd37da1d593431c33a-46',
      'venue': {'categories': [{'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/food/default_',
          'suffix': '.png'},
         'id': '5bae9231bedf3950379f89d4',
         'name': 'Poke Place',
         'pluralName': 'Poke Places',
         'primary': True,
         'shortName': 'Poke Place'}],
       'delivery': {'id': '384664',
        'provider': {'icon': {'name': '/delivery_provider_grubhub_20180129.png',
          'prefix': 'https://fastly.4sqi.net/img/general/cap/',
          'sizes': [40, 50]},
         'name': 'grubhub'},
        'url': 'https://www.grubhub.com/restaurant/makai-poke-co-521-washington-st-hoboken/384664?affiliate=1131&utm_source=foursquare-affiliate-network&utm_medium=affiliate&utm_campaign=1131&utm_content=384664'},
       'id': '58c470fd37da1d593431c33a',
       'location': {'address': '521 Washington St',
        'cc': 'US',
        'city': 'Hoboken',
        'country': 'United States',
        'distance': 271,
        'formattedAddress': ['521 Washington St',
         'Hoboken, NJ 07030',
         'United States'],
        'labeledLatLngs': [{'label': 'display',
          'lat': 40.74285236329213,
          'lng': -74.0292129303126}],
        'lat': 40.74285236329213,
        'lng': -74.0292129303126,
        'postalCode': '07030',
        'state': 'NJ'},
       'name': 'Makai Poke Co',
       'photos': {'count': 0, 'groups': []}}},
     {'reasons': {'count': 0,
       'items': [{'reasonName': 'globalInteractionReason',
         'summary': 'This spot is popular',
         'type': 'general'}]},
      'referralId': 'e-0-4c41e1aee26920a1981e5fe7-47',
      'venue': {'categories': [{'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/food/korean_',
          'suffix': '.png'},
         'id': '4bf58dd8d48988d113941735',
         'name': 'Korean Restaurant',
         'pluralName': 'Korean Restaurants',
         'primary': True,
         'shortName': 'Korean'}],
       'delivery': {'id': '263710',
        'provider': {'icon': {'name': '/delivery_provider_grubhub_20180129.png',
          'prefix': 'https://fastly.4sqi.net/img/general/cap/',
          'sizes': [40, 50]},
         'name': 'grubhub'},
        'url': 'https://www.grubhub.com/restaurant/chicken-factory-529-washington-st-hoboken/263710?affiliate=1131&utm_source=foursquare-affiliate-network&utm_medium=affiliate&utm_campaign=1131&utm_content=263710'},
       'id': '4c41e1aee26920a1981e5fe7',
       'location': {'address': '529 Washington St',
        'cc': 'US',
        'city': 'Hoboken',
        'country': 'United States',
        'distance': 280,
        'formattedAddress': ['529 Washington St',
         'Hoboken, NJ 07030',
         'United States'],
        'labeledLatLngs': [{'label': 'display',
          'lat': 40.742975,
          'lng': -74.029083}],
        'lat': 40.742975,
        'lng': -74.029083,
        'postalCode': '07030',
        'state': 'NJ'},
       'name': 'Chicken Factory',
       'photos': {'count': 0, 'groups': []}}},
     {'reasons': {'count': 0,
       'items': [{'reasonName': 'globalInteractionReason',
         'summary': 'This spot is popular',
         'type': 'general'}]},
      'referralId': 'e-0-4c682b63e1da1b8d4a729fc3-48',
      'venue': {'categories': [{'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/shops/jewelry_',
          'suffix': '.png'},
         'id': '4bf58dd8d48988d111951735',
         'name': 'Jewelry Store',
         'pluralName': 'Jewelry Stores',
         'primary': True,
         'shortName': 'Jewelry'}],
       'id': '4c682b63e1da1b8d4a729fc3',
       'location': {'address': '408 6th St',
        'cc': 'US',
        'city': 'Hoboken',
        'country': 'United States',
        'crossStreet': 'Btwn Grand & Adams',
        'distance': 260,
        'formattedAddress': ['408 6th St (Btwn Grand & Adams)',
         'Hoboken, NJ 07030',
         'United States'],
        'labeledLatLngs': [{'label': 'display',
          'lat': 40.74448674514443,
          'lng': -74.03503853252423}],
        'lat': 40.74448674514443,
        'lng': -74.03503853252423,
        'neighborhood': 'West of Willow - WoW',
        'postalCode': '07030',
        'state': 'NJ'},
       'name': 'aaRaa',
       'photos': {'count': 0, 'groups': []},
       'venuePage': {'id': '40077345'}}},
     {'reasons': {'count': 0,
       'items': [{'reasonName': 'globalInteractionReason',
         'summary': 'This spot is popular',
         'type': 'general'}]},
      'referralId': 'e-0-51070e83e4b0d92d935e04a3-49',
      'venue': {'categories': [{'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/food/deli_',
          'suffix': '.png'},
         'id': '4bf58dd8d48988d1c5941735',
         'name': 'Sandwich Place',
         'pluralName': 'Sandwich Places',
         'primary': True,
         'shortName': 'Sandwiches'}],
       'delivery': {'id': '331114',
        'provider': {'icon': {'name': '/delivery_provider_grubhub_20180129.png',
          'prefix': 'https://fastly.4sqi.net/img/general/cap/',
          'sizes': [40, 50]},
         'name': 'grubhub'},
        'url': 'https://www.grubhub.com/restaurant/midtown-philly-steaks-523-washington-st-hoboken/331114?affiliate=1131&utm_source=foursquare-affiliate-network&utm_medium=affiliate&utm_campaign=1131&utm_content=331114'},
       'id': '51070e83e4b0d92d935e04a3',
       'location': {'address': '523 Washington St',
        'cc': 'US',
        'city': 'Hoboken',
        'country': 'United States',
        'distance': 269,
        'formattedAddress': ['523 Washington St',
         'Hoboken, NJ 07030',
         'United States'],
        'labeledLatLngs': [{'label': 'display',
          'lat': 40.742983164232264,
          'lng': -74.02921097211629}],
        'lat': 40.742983164232264,
        'lng': -74.02921097211629,
        'postalCode': '07030',
        'state': 'NJ'},
       'name': 'Midtown Philly Steaks',
       'photos': {'count': 0, 'groups': []}}},
     {'reasons': {'count': 0,
       'items': [{'reasonName': 'globalInteractionReason',
         'summary': 'This spot is popular',
         'type': 'general'}]},
      'referralId': 'e-0-5a6b6047f427de038c51031c-50',
      'venue': {'categories': [{'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/food/falafel_',
          'suffix': '.png'},
         'id': '4bf58dd8d48988d10b941735',
         'name': 'Falafel Restaurant',
         'pluralName': 'Falafel Restaurants',
         'primary': True,
         'shortName': 'Falafel'}],
       'id': '5a6b6047f427de038c51031c',
       'location': {'address': '300 Washington St',
        'cc': 'US',
        'city': 'Hoboken',
        'country': 'United States',
        'distance': 395,
        'formattedAddress': ['300 Washington St',
         'Hoboken, NJ 07030',
         'United States'],
        'labeledLatLngs': [{'label': 'display',
          'lat': 40.74009035450262,
          'lng': -74.03037432533311}],
        'lat': 40.74009035450262,
        'lng': -74.03037432533311,
        'postalCode': '07030',
        'state': 'NJ'},
       'name': 'Mamoun’s Falafel',
       'photos': {'count': 0, 'groups': []}}},
     {'reasons': {'count': 0,
       'items': [{'reasonName': 'globalInteractionReason',
         'summary': 'This spot is popular',
         'type': 'general'}]},
      'referralId': 'e-0-4ad89c0bf964a520d31221e3-51',
      'venue': {'categories': [{'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/food/default_',
          'suffix': '.png'},
         'id': '4bf58dd8d48988d14e941735',
         'name': 'American Restaurant',
         'pluralName': 'American Restaurants',
         'primary': True,
         'shortName': 'American'}],
       'id': '4ad89c0bf964a520d31221e3',
       'location': {'address': '741 Garden St',
        'cc': 'US',
        'city': 'Hoboken',
        'country': 'United States',
        'distance': 389,
        'formattedAddress': ['741 Garden St',
         'Hoboken, NJ 07030',
         'United States'],
        'labeledLatLngs': [{'label': 'display',
          'lat': 40.746307825161225,
          'lng': -74.03000307300934}],
        'lat': 40.746307825161225,
        'lng': -74.03000307300934,
        'postalCode': '07030',
        'state': 'NJ'},
       'name': "Mr Wrap's",
       'photos': {'count': 0, 'groups': []},
       'venuePage': {'id': '84521331'}}},
     {'reasons': {'count': 0,
       'items': [{'reasonName': 'globalInteractionReason',
         'summary': 'This spot is popular',
         'type': 'general'}]},
      'referralId': 'e-0-4a74aee1f964a5202ddf1fe3-52',
      'venue': {'categories': [{'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/shops/record_shop_',
          'suffix': '.png'},
         'id': '4bf58dd8d48988d10d951735',
         'name': 'Record Shop',
         'pluralName': 'Record Shops',
         'primary': True,
         'shortName': 'Record Shop'}],
       'id': '4a74aee1f964a5202ddf1fe3',
       'location': {'address': '225 Washington St',
        'cc': 'US',
        'city': 'Hoboken',
        'country': 'United States',
        'distance': 472,
        'formattedAddress': ['225 Washington St',
         'Hoboken, NJ 07030',
         'United States'],
        'labeledLatLngs': [{'label': 'display',
          'lat': 40.73942636611954,
          'lng': -74.03009628783576}],
        'lat': 40.73942636611954,
        'lng': -74.03009628783576,
        'postalCode': '07030',
        'state': 'NJ'},
       'name': 'Tunes',
       'photos': {'count': 0, 'groups': []}}},
     {'reasons': {'count': 0,
       'items': [{'reasonName': 'globalInteractionReason',
         'summary': 'This spot is popular',
         'type': 'general'}]},
      'referralId': 'e-0-4a775512f964a5202ee41fe3-53',
      'venue': {'categories': [{'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/food/italian_',
          'suffix': '.png'},
         'id': '4bf58dd8d48988d110941735',
         'name': 'Italian Restaurant',
         'pluralName': 'Italian Restaurants',
         'primary': True,
         'shortName': 'Italian'}],
       'delivery': {'id': '773233',
        'provider': {'icon': {'name': '/delivery_provider_grubhub_20180129.png',
          'prefix': 'https://fastly.4sqi.net/img/general/cap/',
          'sizes': [40, 50]},
         'name': 'grubhub'},
        'url': 'https://www.grubhub.com/restaurant/margheritas-restaurant-740-washington-st-hoboken/773233?affiliate=1131&utm_source=foursquare-affiliate-network&utm_medium=affiliate&utm_campaign=1131&utm_content=773233'},
       'id': '4a775512f964a5202ee41fe3',
       'location': {'address': '740 Washington St',
        'cc': 'US',
        'city': 'Hoboken',
        'country': 'United States',
        'crossStreet': 'at 8th St.',
        'distance': 451,
        'formattedAddress': ['740 Washington St (at 8th St.)',
         'Hoboken, NJ 07030',
         'United States'],
        'labeledLatLngs': [{'label': 'display',
          'lat': 40.746033508515865,
          'lng': -74.02841202436005}],
        'lat': 40.746033508515865,
        'lng': -74.02841202436005,
        'postalCode': '07030',
        'state': 'NJ'},
       'name': "Margherita's",
       'photos': {'count': 0, 'groups': []}}},
     {'reasons': {'count': 0,
       'items': [{'reasonName': 'globalInteractionReason',
         'summary': 'This spot is popular',
         'type': 'general'}]},
      'referralId': 'e-0-4a3ad481f964a52057a01fe3-54',
      'venue': {'categories': [{'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/nightlife/sportsbar_',
          'suffix': '.png'},
         'id': '4bf58dd8d48988d11d941735',
         'name': 'Sports Bar',
         'pluralName': 'Sports Bars',
         'primary': True,
         'shortName': 'Sports Bar'}],
       'delivery': {'id': '277105',
        'provider': {'icon': {'name': '/delivery_provider_grubhub_20180129.png',
          'prefix': 'https://fastly.4sqi.net/img/general/cap/',
          'sizes': [40, 50]},
         'name': 'grubhub'},
        'url': 'https://www.grubhub.com/restaurant/mikie-squared-bar--grill-616-washington-st--hoboken/277105?affiliate=1131&utm_source=foursquare-affiliate-network&utm_medium=affiliate&utm_campaign=1131&utm_content=277105'},
       'id': '4a3ad481f964a52057a01fe3',
       'location': {'address': '616 Washington St',
        'cc': 'US',
        'city': 'Hoboken',
        'country': 'United States',
        'crossStreet': 'btwn 6th & 7th',
        'distance': 299,
        'formattedAddress': ['616 Washington St (btwn 6th & 7th)',
         'Hoboken, NJ 07030',
         'United States'],
        'labeledLatLngs': [{'label': 'display',
          'lat': 40.74403210541407,
          'lng': -74.02895604032052}],
        'lat': 40.74403210541407,
        'lng': -74.02895604032052,
        'postalCode': '07030',
        'state': 'NJ'},
       'name': 'Mikie Squared Bar & Grill',
       'photos': {'count': 0, 'groups': []},
       'venuePage': {'id': '375011250'}}},
     {'reasons': {'count': 0,
       'items': [{'reasonName': 'globalInteractionReason',
         'summary': 'This spot is popular',
         'type': 'general'}]},
      'referralId': 'e-0-56cb57c7cd10aa9184195eec-55',
      'venue': {'categories': [{'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/food/juicebar_',
          'suffix': '.png'},
         'id': '4bf58dd8d48988d112941735',
         'name': 'Juice Bar',
         'pluralName': 'Juice Bars',
         'primary': True,
         'shortName': 'Juice Bar'}],
       'id': '56cb57c7cd10aa9184195eec',
       'location': {'address': '322 Washington St',
        'cc': 'US',
        'city': 'Hoboken',
        'country': 'United States',
        'distance': 340,
        'formattedAddress': ['322 Washington St',
         'Hoboken, NJ 07030',
         'United States'],
        'labeledLatLngs': [{'label': 'display',
          'lat': 40.74089232991324,
          'lng': -74.02989603456226}],
        'lat': 40.74089232991324,
        'lng': -74.02989603456226,
        'postalCode': '07030',
        'state': 'NJ'},
       'name': 'Total Nutrition Kitchen',
       'photos': {'count': 0, 'groups': []}}},
     {'reasons': {'count': 0,
       'items': [{'reasonName': 'globalInteractionReason',
         'summary': 'This spot is popular',
         'type': 'general'}]},
      'referralId': 'e-0-4c32368b3896e21eba28e890-56',
      'venue': {'categories': [{'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/shops/beauty_cosmetic_',
          'suffix': '.png'},
         'id': '4bf58dd8d48988d10c951735',
         'name': 'Cosmetics Shop',
         'pluralName': 'Cosmetics Shops',
         'primary': True,
         'shortName': 'Cosmetics'}],
       'id': '4c32368b3896e21eba28e890',
       'location': {'address': '728 Washington St',
        'cc': 'US',
        'city': 'Hoboken',
        'country': 'United States',
        'distance': 416,
        'formattedAddress': ['728 Washington St',
         'Hoboken, NJ 07030',
         'United States'],
        'labeledLatLngs': [{'label': 'display',
          'lat': 40.74562444483551,
          'lng': -74.02850120376685}],
        'lat': 40.74562444483551,
        'lng': -74.02850120376685,
        'postalCode': '07030',
        'state': 'NJ'},
       'name': 'Modern Nails and Spa',
       'photos': {'count': 0, 'groups': []}}},
     {'reasons': {'count': 0,
       'items': [{'reasonName': 'globalInteractionReason',
         'summary': 'This spot is popular',
         'type': 'general'}]},
      'referralId': 'e-0-5abbff66123a1973728506b7-57',
      'venue': {'categories': [{'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/shops/beauty_cosmetic_',
          'suffix': '.png'},
         'id': '4bf58dd8d48988d10c951735',
         'name': 'Cosmetics Shop',
         'pluralName': 'Cosmetics Shops',
         'primary': True,
         'shortName': 'Cosmetics'}],
       'id': '5abbff66123a1973728506b7',
       'location': {'address': '319 Washington St.',
        'cc': 'US',
        'city': 'Hoboken',
        'country': 'United States',
        'distance': 377,
        'formattedAddress': ['319 Washington St.',
         'Hoboken, NJ 07030',
         'United States'],
        'labeledLatLngs': [{'label': 'display',
          'lat': 40.740584640857804,
          'lng': -74.02971545155526}],
        'lat': 40.740584640857804,
        'lng': -74.02971545155526,
        'postalCode': '07030',
        'state': 'NJ'},
       'name': 'SEPHORA',
       'photos': {'count': 0, 'groups': []}}},
     {'reasons': {'count': 0,
       'items': [{'reasonName': 'globalInteractionReason',
         'summary': 'This spot is popular',
         'type': 'general'}]},
      'referralId': 'e-0-4a9703def964a520fb2720e3-58',
      'venue': {'categories': [{'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/food/tearoom_',
          'suffix': '.png'},
         'id': '4bf58dd8d48988d1dc931735',
         'name': 'Tea Room',
         'pluralName': 'Tea Rooms',
         'primary': True,
         'shortName': 'Tea Room'}],
       'id': '4a9703def964a520fb2720e3',
       'location': {'address': '638 Willow Ave',
        'cc': 'US',
        'city': 'Hoboken',
        'country': 'United States',
        'crossStreet': '7th St',
        'distance': 222,
        'formattedAddress': ['638 Willow Ave (7th St)',
         'Hoboken, NJ 07030',
         'United States'],
        'labeledLatLngs': [{'label': 'display',
          'lat': 40.74530714988045,
          'lng': -74.03235873725076}],
        'lat': 40.74530714988045,
        'lng': -74.03235873725076,
        'postalCode': '07030',
        'state': 'NJ'},
       'name': 'Maroon',
       'photos': {'count': 0, 'groups': []}}},
     {'reasons': {'count': 0,
       'items': [{'reasonName': 'globalInteractionReason',
         'summary': 'This spot is popular',
         'type': 'general'}]},
      'referralId': 'e-0-5065c556e4b0a44a76b324ce-59',
      'venue': {'categories': [{'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/food/mexican_',
          'suffix': '.png'},
         'id': '4bf58dd8d48988d1c1941735',
         'name': 'Mexican Restaurant',
         'pluralName': 'Mexican Restaurants',
         'primary': True,
         'shortName': 'Mexican'}],
       'id': '5065c556e4b0a44a76b324ce',
       'location': {'address': '229 Washington St',
        'cc': 'US',
        'city': 'Hoboken',
        'country': 'United States',
        'crossStreet': 'Washington St and 3rd St',
        'distance': 459,
        'formattedAddress': ['229 Washington St (Washington St and 3rd St)',
         'Hoboken, NJ 07030',
         'United States'],
        'labeledLatLngs': [{'label': 'display',
          'lat': 40.739599465749336,
          'lng': -74.02998078583823}],
        'lat': 40.739599465749336,
        'lng': -74.02998078583823,
        'postalCode': '07030',
        'state': 'NJ'},
       'name': 'Chipotle Mexican Grill',
       'photos': {'count': 0, 'groups': []}}},
     {'reasons': {'count': 0,
       'items': [{'reasonName': 'globalInteractionReason',
         'summary': 'This spot is popular',
         'type': 'general'}]},
      'referralId': 'e-0-49e37784f964a52083621fe3-60',
      'venue': {'categories': [{'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/food/mexican_',
          'suffix': '.png'},
         'id': '4bf58dd8d48988d1c1941735',
         'name': 'Mexican Restaurant',
         'pluralName': 'Mexican Restaurants',
         'primary': True,
         'shortName': 'Mexican'}],
       'delivery': {'id': '306879',
        'provider': {'icon': {'name': '/delivery_provider_grubhub_20180129.png',
          'prefix': 'https://fastly.4sqi.net/img/general/cap/',
          'sizes': [40, 50]},
         'name': 'grubhub'},
        'url': 'https://www.grubhub.com/restaurant/east-la-508-washington-st-hoboken/306879?affiliate=1131&utm_source=foursquare-affiliate-network&utm_medium=affiliate&utm_campaign=1131&utm_content=306879'},
       'id': '49e37784f964a52083621fe3',
       'location': {'address': '508 Washington St',
        'cc': 'US',
        'city': 'Hoboken',
        'country': 'United States',
        'crossStreet': '5th St',
        'distance': 258,
        'formattedAddress': ['508 Washington St (5th St)',
         'Hoboken, NJ 07030',
         'United States'],
        'labeledLatLngs': [{'label': 'display',
          'lat': 40.742508967942605,
          'lng': -74.0294958940094}],
        'lat': 40.742508967942605,
        'lng': -74.0294958940094,
        'postalCode': '07030',
        'state': 'NJ'},
       'name': 'East LA',
       'photos': {'count': 0, 'groups': []},
       'venuePage': {'id': '43154962'}}},
     {'reasons': {'count': 0,
       'items': [{'reasonName': 'globalInteractionReason',
         'summary': 'This spot is popular',
         'type': 'general'}]},
      'referralId': 'e-0-5131669e582f0b9471213351-61',
      'venue': {'categories': [{'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/nightlife/pub_',
          'suffix': '.png'},
         'id': '4bf58dd8d48988d116941735',
         'name': 'Bar',
         'pluralName': 'Bars',
         'primary': True,
         'shortName': 'Bar'}],
       'delivery': {'id': '287703',
        'provider': {'icon': {'name': '/delivery_provider_grubhub_20180129.png',
          'prefix': 'https://fastly.4sqi.net/img/general/cap/',
          'sizes': [40, 50]},
         'name': 'grubhub'},
        'url': 'https://www.grubhub.com/restaurant/the-stewed-cow-400-adams-st-hoboken/287703?affiliate=1131&utm_source=foursquare-affiliate-network&utm_medium=affiliate&utm_campaign=1131&utm_content=287703'},
       'id': '5131669e582f0b9471213351',
       'location': {'address': '400 Adams St',
        'cc': 'US',
        'city': 'Hoboken',
        'country': 'United States',
        'crossStreet': '4th St',
        'distance': 351,
        'formattedAddress': ['400 Adams St (4th St)',
         'Hoboken, NJ 07030',
         'United States'],
        'labeledLatLngs': [{'label': 'display',
          'lat': 40.74246053838308,
          'lng': -74.03638450154322}],
        'lat': 40.74246053838308,
        'lng': -74.03638450154322,
        'postalCode': '07030',
        'state': 'NJ'},
       'name': 'The Stewed Cow',
       'photos': {'count': 0, 'groups': []},
       'venuePage': {'id': '56376141'}}},
     {'reasons': {'count': 0,
       'items': [{'reasonName': 'globalInteractionReason',
         'summary': 'This spot is popular',
         'type': 'general'}]},
      'referralId': 'e-0-4bc3bdc2920eb71334f31d2c-62',
      'venue': {'categories': [{'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/shops/beauty_cosmetic_',
          'suffix': '.png'},
         'id': '4bf58dd8d48988d10c951735',
         'name': 'Cosmetics Shop',
         'pluralName': 'Cosmetics Shops',
         'primary': True,
         'shortName': 'Cosmetics'}],
       'id': '4bc3bdc2920eb71334f31d2c',
       'location': {'address': '302 Washington St',
        'cc': 'US',
        'city': 'Hoboken',
        'country': 'United States',
        'crossStreet': 'at 3rd St.',
        'distance': 405,
        'formattedAddress': ['302 Washington St (at 3rd St.)',
         'Hoboken, NJ 07030',
         'United States'],
        'labeledLatLngs': [{'label': 'display',
          'lat': 40.74007790019808,
          'lng': -74.03016149463437}],
        'lat': 40.74007790019808,
        'lng': -74.03016149463437,
        'postalCode': '07030',
        'state': 'NJ'},
       'name': 'Makeover II',
       'photos': {'count': 0, 'groups': []}}},
     {'reasons': {'count': 0,
       'items': [{'reasonName': 'globalInteractionReason',
         'summary': 'This spot is popular',
         'type': 'general'}]},
      'referralId': 'e-0-4c928177418ea1cd8c86a585-63',
      'venue': {'categories': [{'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/building/default_',
          'suffix': '.png'},
         'id': '5453de49498eade8af355881',
         'name': 'Business Service',
         'pluralName': 'Business Services',
         'primary': True,
         'shortName': 'Business Services'}],
       'id': '4c928177418ea1cd8c86a585',
       'location': {'address': '701 Grand St',
        'cc': 'US',
        'city': 'Hoboken',
        'country': 'United States',
        'crossStreet': '7th st',
        'distance': 312,
        'formattedAddress': ['701 Grand St (7th st)',
         'Hoboken, NJ 07030',
         'United States'],
        'labeledLatLngs': [{'label': 'display',
          'lat': 40.74584265387637,
          'lng': -74.03396844863892}],
        'lat': 40.74584265387637,
        'lng': -74.03396844863892,
        'postalCode': '07030',
        'state': 'NJ'},
       'name': 'eMazzanti Technologies',
       'photos': {'count': 0, 'groups': []},
       'venuePage': {'id': '60923283'}}},
     {'reasons': {'count': 0,
       'items': [{'reasonName': 'globalInteractionReason',
         'summary': 'This spot is popular',
         'type': 'general'}]},
      'referralId': 'e-0-55d52901498ea18f871d5f9e-64',
      'venue': {'categories': [{'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/food/default_',
          'suffix': '.png'},
         'id': '4bf58dd8d48988d1c4941735',
         'name': 'Restaurant',
         'pluralName': 'Restaurants',
         'primary': True,
         'shortName': 'Restaurant'}],
       'delivery': {'id': '315593',
        'provider': {'icon': {'name': '/delivery_provider_grubhub_20180129.png',
          'prefix': 'https://fastly.4sqi.net/img/general/cap/',
          'sizes': [40, 50]},
         'name': 'grubhub'},
        'url': 'https://www.grubhub.com/restaurant/flatbread-grill-517-washington-st-hoboken/315593?affiliate=1131&utm_source=foursquare-affiliate-network&utm_medium=affiliate&utm_campaign=1131&utm_content=315593'},
       'id': '55d52901498ea18f871d5f9e',
       'location': {'address': '517 Washington St',
        'cc': 'US',
        'city': 'Hoboken',
        'country': 'United States',
        'distance': 267,
        'formattedAddress': ['517 Washington St',
         'Hoboken, NJ 07030',
         'United States'],
        'labeledLatLngs': [{'label': 'display',
          'lat': 40.74276678231329,
          'lng': -74.0292813999794}],
        'lat': 40.74276678231329,
        'lng': -74.0292813999794,
        'postalCode': '07030',
        'state': 'NJ'},
       'name': 'Flatbread Grill',
       'photos': {'count': 0, 'groups': []}}},
     {'reasons': {'count': 0,
       'items': [{'reasonName': 'globalInteractionReason',
         'summary': 'This spot is popular',
         'type': 'general'}]},
      'referralId': 'e-0-4b191a58f964a520ffd723e3-65',
      'venue': {'categories': [{'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/food/donuts_',
          'suffix': '.png'},
         'id': '4bf58dd8d48988d148941735',
         'name': 'Donut Shop',
         'pluralName': 'Donut Shops',
         'primary': True,
         'shortName': 'Donuts'}],
       'id': '4b191a58f964a520ffd723e3',
       'location': {'address': '700 Washington St',
        'cc': 'US',
        'city': 'Hoboken',
        'country': 'United States',
        'crossStreet': 'at 7th St.',
        'distance': 344,
        'formattedAddress': ['700 Washington St (at 7th St.)',
         'Hoboken, NJ 07030',
         'United States'],
        'labeledLatLngs': [{'label': 'display',
          'lat': 40.744813205698065,
          'lng': -74.02881198461071}],
        'lat': 40.744813205698065,
        'lng': -74.02881198461071,
        'postalCode': '07030',
        'state': 'NJ'},
       'name': "Dunkin' Donuts",
       'photos': {'count': 0, 'groups': []}}},
     {'reasons': {'count': 0,
       'items': [{'reasonName': 'globalInteractionReason',
         'summary': 'This spot is popular',
         'type': 'general'}]},
      'referralId': 'e-0-56f136ad498ecc5661aa49ce-66',
      'venue': {'categories': [{'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/food/deli_',
          'suffix': '.png'},
         'id': '4bf58dd8d48988d1c5941735',
         'name': 'Sandwich Place',
         'pluralName': 'Sandwich Places',
         'primary': True,
         'shortName': 'Sandwiches'}],
       'delivery': {'id': '325317',
        'provider': {'icon': {'name': '/delivery_provider_grubhub_20180129.png',
          'prefix': 'https://fastly.4sqi.net/img/general/cap/',
          'sizes': [40, 50]},
         'name': 'grubhub'},
        'url': 'https://www.grubhub.com/restaurant/pita-pit-732-washington-st-hoboken/325317?affiliate=1131&utm_source=foursquare-affiliate-network&utm_medium=affiliate&utm_campaign=1131&utm_content=325317'},
       'id': '56f136ad498ecc5661aa49ce',
       'location': {'address': '732 Washington St, Apt 1',
        'cc': 'US',
        'city': 'Hoboken',
        'country': 'United States',
        'distance': 418,
        'formattedAddress': ['732 Washington St, Apt 1',
         'Hoboken, NJ 07030',
         'United States'],
        'labeledLatLngs': [{'label': 'display',
          'lat': 40.745729,
          'lng': -74.028582}],
        'lat': 40.745729,
        'lng': -74.028582,
        'postalCode': '07030',
        'state': 'NJ'},
       'name': 'Pita Pit',
       'photos': {'count': 0, 'groups': []}}},
     {'reasons': {'count': 0,
       'items': [{'reasonName': 'globalInteractionReason',
         'summary': 'This spot is popular',
         'type': 'general'}]},
      'referralId': 'e-0-4b79c7d0f964a520a1112fe3-67',
      'venue': {'categories': [{'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/shops/pet_store_',
          'suffix': '.png'},
         'id': '4bf58dd8d48988d100951735',
         'name': 'Pet Store',
         'pluralName': 'Pet Stores',
         'primary': True,
         'shortName': 'Pet Store'}],
       'id': '4b79c7d0f964a520a1112fe3',
       'location': {'address': '106 5th St',
        'cc': 'US',
        'city': 'Hoboken',
        'country': 'United States',
        'crossStreet': 'btwn Washington & Bloomfield',
        'distance': 238,
        'formattedAddress': ['106 5th St (btwn Washington & Bloomfield)',
         'Hoboken, NJ 07030',
         'United States'],
        'labeledLatLngs': [{'label': 'display',
          'lat': 40.7422716423953,
          'lng': -74.02989590123302}],
        'lat': 40.7422716423953,
        'lng': -74.02989590123302,
        'postalCode': '07030',
        'state': 'NJ'},
       'name': 'Beowoof',
       'photos': {'count': 0, 'groups': []}}},
     {'reasons': {'count': 0,
       'items': [{'reasonName': 'globalInteractionReason',
         'summary': 'This spot is popular',
         'type': 'general'}]},
      'referralId': 'e-0-4bc1df1d2a89ef3bc1cdf288-68',
      'venue': {'categories': [{'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/shops/salon_barber_',
          'suffix': '.png'},
         'id': '4bf58dd8d48988d110951735',
         'name': 'Salon / Barbershop',
         'pluralName': 'Salons / Barbershops',
         'primary': True,
         'shortName': 'Salon / Barbershop'}],
       'id': '4bc1df1d2a89ef3bc1cdf288',
       'location': {'address': '402 Washington St',
        'cc': 'US',
        'city': 'Hoboken',
        'country': 'United States',
        'crossStreet': '4th St',
        'distance': 308,
        'formattedAddress': ['402 Washington St (4th St)',
         'Hoboken, NJ 07030',
         'United States'],
        'labeledLatLngs': [{'label': 'display',
          'lat': 40.74139329755372,
          'lng': -74.02972348211598}],
        'lat': 40.74139329755372,
        'lng': -74.02972348211598,
        'postalCode': '07030',
        'state': 'NJ'},
       'name': 'Bloom Spa And Nails',
       'photos': {'count': 0, 'groups': []}}},
     {'reasons': {'count': 0,
       'items': [{'reasonName': 'globalInteractionReason',
         'summary': 'This spot is popular',
         'type': 'general'}]},
      'referralId': 'e-0-4e4d8d3eae608af796ea0e86-69',
      'venue': {'categories': [{'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/shops/food_liquor_',
          'suffix': '.png'},
         'id': '4bf58dd8d48988d186941735',
         'name': 'Liquor Store',
         'pluralName': 'Liquor Stores',
         'primary': True,
         'shortName': 'Liquor Store'}],
       'id': '4e4d8d3eae608af796ea0e86',
       'location': {'address': '528 Adams St',
        'cc': 'US',
        'city': 'Hoboken',
        'country': 'United States',
        'distance': 321,
        'formattedAddress': ['528 Adams St',
         'Hoboken, NJ 07030',
         'United States'],
        'labeledLatLngs': [{'label': 'display',
          'lat': 40.74392239995025,
          'lng': -74.03610202954108}],
        'lat': 40.74392239995025,
        'lng': -74.03610202954108,
        'postalCode': '07030',
        'state': 'NJ'},
       'name': "Caporrino's News & Liquor",
       'photos': {'count': 0, 'groups': []},
       'venuePage': {'id': '84955725'}}},
     {'reasons': {'count': 0,
       'items': [{'reasonName': 'globalInteractionReason',
         'summary': 'This spot is popular',
         'type': 'general'}]},
      'referralId': 'e-0-5ac2cec0419a9e0f679671f3-70',
      'venue': {'categories': [{'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/food/deli_',
          'suffix': '.png'},
         'id': '4bf58dd8d48988d1c5941735',
         'name': 'Sandwich Place',
         'pluralName': 'Sandwich Places',
         'primary': True,
         'shortName': 'Sandwiches'}],
       'delivery': {'id': '754203',
        'provider': {'icon': {'name': '/delivery_provider_grubhub_20180129.png',
          'prefix': 'https://fastly.4sqi.net/img/general/cap/',
          'sizes': [40, 50]},
         'name': 'grubhub'},
        'url': 'https://www.grubhub.com/restaurant/sauced-217-washington-st-hoboken/754203?affiliate=1131&utm_source=foursquare-affiliate-network&utm_medium=affiliate&utm_campaign=1131&utm_content=754203'},
       'id': '5ac2cec0419a9e0f679671f3',
       'location': {'address': '217 Washington St',
        'cc': 'US',
        'city': 'Hoboken',
        'country': 'United States',
        'distance': 484,
        'formattedAddress': ['217 Washington St',
         'Hoboken, NJ 07030',
         'United States'],
        'labeledLatLngs': [{'label': 'display',
          'lat': 40.73927400056595,
          'lng': -74.03020475686708}],
        'lat': 40.73927400056595,
        'lng': -74.03020475686708,
        'postalCode': '07030',
        'state': 'NJ'},
       'name': 'Sauced',
       'photos': {'count': 0, 'groups': []}}},
     {'reasons': {'count': 0,
       'items': [{'reasonName': 'globalInteractionReason',
         'summary': 'This spot is popular',
         'type': 'general'}]},
      'referralId': 'e-0-4b035ae1f964a520e24e22e3-71',
      'venue': {'categories': [{'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/shops/food_grocery_',
          'suffix': '.png'},
         'id': '4bf58dd8d48988d118951735',
         'name': 'Grocery Store',
         'pluralName': 'Grocery Stores',
         'primary': True,
         'shortName': 'Grocery Store'}],
       'id': '4b035ae1f964a520e24e22e3',
       'location': {'address': '204 Washington St',
        'cc': 'US',
        'city': 'Hoboken',
        'country': 'United States',
        'crossStreet': 'btwn 2nd & 3rd',
        'distance': 500,
        'formattedAddress': ['204 Washington St (btwn 2nd & 3rd)',
         'Hoboken, NJ 07030',
         'United States'],
        'labeledLatLngs': [{'label': 'display',
          'lat': 40.739068266966825,
          'lng': -74.03038849034185}],
        'lat': 40.739068266966825,
        'lng': -74.03038849034185,
        'postalCode': '07030',
        'state': 'NJ'},
       'name': 'Basic Food',
       'photos': {'count': 0, 'groups': []}}},
     {'reasons': {'count': 0,
       'items': [{'reasonName': 'globalInteractionReason',
         'summary': 'This spot is popular',
         'type': 'general'}]},
      'referralId': 'e-0-4e3d3837b0fb875af85e8b72-72',
      'venue': {'categories': [{'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/shops/pet_store_',
          'suffix': '.png'},
         'id': '4bf58dd8d48988d100951735',
         'name': 'Pet Store',
         'pluralName': 'Pet Stores',
         'primary': True,
         'shortName': 'Pet Store'}],
       'id': '4e3d3837b0fb875af85e8b72',
       'location': {'address': '508 5th St',
        'cc': 'US',
        'city': 'Hoboken',
        'country': 'United States',
        'distance': 407,
        'formattedAddress': ['508 5th St',
         'Hoboken, NJ 07030',
         'United States'],
        'labeledLatLngs': [{'label': 'display',
          'lat': 40.74362214778684,
          'lng': -74.03718642116372}],
        'lat': 40.74362214778684,
        'lng': -74.03718642116372,
        'postalCode': '07030',
        'state': 'NJ'},
       'name': 'Cozy Cuts Pets Grooming, LLC',
       'photos': {'count': 0, 'groups': []}}},
     {'reasons': {'count': 0,
       'items': [{'reasonName': 'globalInteractionReason',
         'summary': 'This spot is popular',
         'type': 'general'}]},
      'referralId': 'e-0-562c00b2498eb20825dcc6c0-73',
      'venue': {'categories': [{'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/shops/default_',
          'suffix': '.png'},
         'id': '52f2ab2ebcbc57f1066b8b21',
         'name': 'Stationery Store',
         'pluralName': 'Stationery Stores',
         'primary': True,
         'shortName': 'Stationery Store'}],
       'id': '562c00b2498eb20825dcc6c0',
       'location': {'address': '312 Washington St',
        'cc': 'US',
        'city': 'Hoboken',
        'country': 'United States',
        'distance': 365,
        'formattedAddress': ['312 Washington St',
         'Hoboken, NJ 07030',
         'United States'],
        'labeledLatLngs': [{'label': 'display',
          'lat': 40.74053961954923,
          'lng': -74.03004467344562}],
        'lat': 40.74053961954923,
        'lng': -74.03004467344562,
        'postalCode': '07030',
        'state': 'NJ'},
       'name': 'Hudson Paperie',
       'photos': {'count': 0, 'groups': []}}},
     {'reasons': {'count': 0,
       'items': [{'reasonName': 'globalInteractionReason',
         'summary': 'This spot is popular',
         'type': 'general'}]},
      'referralId': 'e-0-49e27ed2f964a5201c621fe3-74',
      'venue': {'categories': [{'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/food/thai_',
          'suffix': '.png'},
         'id': '4bf58dd8d48988d149941735',
         'name': 'Thai Restaurant',
         'pluralName': 'Thai Restaurants',
         'primary': True,
         'shortName': 'Thai'}],
       'delivery': {'id': '70940',
        'provider': {'icon': {'name': '/delivery_provider_grubhub_20180129.png',
          'prefix': 'https://fastly.4sqi.net/img/general/cap/',
          'sizes': [40, 50]},
         'name': 'grubhub'},
        'url': 'https://www.grubhub.com/restaurant/sri-thai-234-bloomfield-st-hoboken/70940?affiliate=1131&utm_source=foursquare-affiliate-network&utm_medium=affiliate&utm_campaign=1131&utm_content=70940'},
       'id': '49e27ed2f964a5201c621fe3',
       'location': {'address': '234 Bloomfield St',
        'cc': 'US',
        'city': 'Hoboken',
        'country': 'United States',
        'crossStreet': 'at 3rd St.',
        'distance': 376,
        'formattedAddress': ['234 Bloomfield St (at 3rd St.)',
         'Hoboken, NJ 07030',
         'United States'],
        'labeledLatLngs': [{'label': 'display',
          'lat': 40.74010297413233,
          'lng': -74.0309549479617}],
        'lat': 40.74010297413233,
        'lng': -74.0309549479617,
        'postalCode': '07030',
        'state': 'NJ'},
       'name': 'Sri Thai',
       'photos': {'count': 0, 'groups': []}}},
     {'reasons': {'count': 0,
       'items': [{'reasonName': 'globalInteractionReason',
         'summary': 'This spot is popular',
         'type': 'general'}]},
      'referralId': 'e-0-59fd168ec21cb1401894e47e-75',
      'venue': {'categories': [{'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/food/pizza_',
          'suffix': '.png'},
         'id': '4bf58dd8d48988d1ca941735',
         'name': 'Pizza Place',
         'pluralName': 'Pizza Places',
         'primary': True,
         'shortName': 'Pizza'}],
       'id': '59fd168ec21cb1401894e47e',
       'location': {'address': '133 Clinton St',
        'cc': 'US',
        'city': 'Hoboken',
        'country': 'United States',
        'distance': 484,
        'formattedAddress': ['133 Clinton St',
         'Hoboken, NJ 07030',
         'United States'],
        'labeledLatLngs': [{'label': 'display',
          'lat': 40.7394035324866,
          'lng': -74.03490912516594}],
        'lat': 40.7394035324866,
        'lng': -74.03490912516594,
        'postalCode': '07030',
        'state': 'NJ'},
       'name': 'Napoli’s Pizzeria',
       'photos': {'count': 0, 'groups': []}}},
     {'reasons': {'count': 0,
       'items': [{'reasonName': 'globalInteractionReason',
         'summary': 'This spot is popular',
         'type': 'general'}]},
      'referralId': 'e-0-4b76e28df964a52021672ee3-76',
      'venue': {'categories': [{'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/food/bagels_',
          'suffix': '.png'},
         'id': '4bf58dd8d48988d179941735',
         'name': 'Bagel Shop',
         'pluralName': 'Bagel Shops',
         'primary': True,
         'shortName': 'Bagels'}],
       'id': '4b76e28df964a52021672ee3',
       'location': {'address': '634 Washington St',
        'cc': 'US',
        'city': 'Hoboken',
        'country': 'United States',
        'crossStreet': 'btwn 6th & 7th',
        'distance': 325,
        'formattedAddress': ['634 Washington St (btwn 6th & 7th)',
         'Hoboken, NJ 07030',
         'United States'],
        'labeledLatLngs': [{'label': 'display',
          'lat': 40.744447667679495,
          'lng': -74.02882241752896}],
        'lat': 40.744447667679495,
        'lng': -74.02882241752896,
        'postalCode': '07030',
        'state': 'NJ'},
       'name': 'Hoboken Hot Bagels',
       'photos': {'count': 0, 'groups': []}}},
     {'reasons': {'count': 0,
       'items': [{'reasonName': 'globalInteractionReason',
         'summary': 'This spot is popular',
         'type': 'general'}]},
      'referralId': 'e-0-4b800a9ef964a520144d30e3-77',
      'venue': {'categories': [{'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/shops/gym_yogastudio_',
          'suffix': '.png'},
         'id': '5744ccdfe4b0c0459246b4b2',
         'name': 'Pilates Studio',
         'pluralName': 'Pilates Studios',
         'primary': True,
         'shortName': 'Pilates Studio'}],
       'id': '4b800a9ef964a520144d30e3',
       'location': {'address': '335 River Street',
        'cc': 'US',
        'city': 'Hoboken',
        'country': 'United States',
        'distance': 495,
        'formattedAddress': ['335 River Street',
         'Hoboken, NJ 07030',
         'United States'],
        'labeledLatLngs': [{'label': 'display',
          'lat': 40.74046006866811,
          'lng': -74.02785846165564}],
        'lat': 40.74046006866811,
        'lng': -74.02785846165564,
        'postalCode': '07030',
        'state': 'NJ'},
       'name': 'Renaissance Pilates',
       'photos': {'count': 0, 'groups': []},
       'venuePage': {'id': '50473125'}}},
     {'reasons': {'count': 0,
       'items': [{'reasonName': 'globalInteractionReason',
         'summary': 'This spot is popular',
         'type': 'general'}]},
      'referralId': 'e-0-5b5b1e0bd4cc9800394e3256-78',
      'venue': {'categories': [{'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/food/coffeeshop_',
          'suffix': '.png'},
         'id': '4bf58dd8d48988d1e0931735',
         'name': 'Coffee Shop',
         'pluralName': 'Coffee Shops',
         'primary': True,
         'shortName': 'Coffee Shop'}],
       'id': '5b5b1e0bd4cc9800394e3256',
       'location': {'address': '700 Garden St',
        'cc': 'US',
        'city': 'Hoboken',
        'country': 'United States',
        'crossStreet': '8th Street',
        'distance': 256,
        'formattedAddress': ['700 Garden St (8th Street)',
         'Hoboken, NJ 07030',
         'United States'],
        'labeledLatLngs': [{'label': 'display',
          'lat': 40.74511745250098,
          'lng': -74.03049325297555}],
        'lat': 40.74511745250098,
        'lng': -74.03049325297555,
        'postalCode': '07030',
        'state': 'NJ'},
       'name': 'Hidden Grounds Coffee',
       'photos': {'count': 0, 'groups': []}}},
     {'reasons': {'count': 0,
       'items': [{'reasonName': 'globalInteractionReason',
         'summary': 'This spot is popular',
         'type': 'general'}]},
      'referralId': 'e-0-4283ee00f964a52091221fe3-79',
      'venue': {'categories': [{'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/nightlife/divebar_',
          'suffix': '.png'},
         'id': '4bf58dd8d48988d118941735',
         'name': 'Dive Bar',
         'pluralName': 'Dive Bars',
         'primary': True,
         'shortName': 'Dive Bar'}],
       'id': '4283ee00f964a52091221fe3',
       'location': {'address': '329 Washington St',
        'cc': 'US',
        'city': 'Hoboken',
        'country': 'United States',
        'crossStreet': 'at 5th St.',
        'distance': 364,
        'formattedAddress': ['329 Washington St (at 5th St.)',
         'Hoboken, NJ 07030',
         'United States'],
        'labeledLatLngs': [{'label': 'display',
          'lat': 40.74088,
          'lng': -74.02948}],
        'lat': 40.74088,
        'lng': -74.02948,
        'postalCode': '07030',
        'state': 'NJ'},
       'name': "Louise & Jerry's",
       'photos': {'count': 0, 'groups': []}}},
     {'reasons': {'count': 0,
       'items': [{'reasonName': 'globalInteractionReason',
         'summary': 'This spot is popular',
         'type': 'general'}]},
      'referralId': 'e-0-4acce0aaf964a520d8c920e3-80',
      'venue': {'categories': [{'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/food/thai_',
          'suffix': '.png'},
         'id': '4bf58dd8d48988d149941735',
         'name': 'Thai Restaurant',
         'pluralName': 'Thai Restaurants',
         'primary': True,
         'shortName': 'Thai'}],
       'delivery': {'id': '260862',
        'provider': {'icon': {'name': '/delivery_provider_grubhub_20180129.png',
          'prefix': 'https://fastly.4sqi.net/img/general/cap/',
          'sizes': [40, 50]},
         'name': 'grubhub'},
        'url': 'https://www.grubhub.com/restaurant/bangkok-city-thai-restaurant-335-washington-st-hoboken/260862?affiliate=1131&utm_source=foursquare-affiliate-network&utm_medium=affiliate&utm_campaign=1131&utm_content=260862'},
       'id': '4acce0aaf964a520d8c920e3',
       'location': {'address': '335 Washington St',
        'cc': 'US',
        'city': 'Hoboken',
        'country': 'United States',
        'crossStreet': 'btw 3rd & 4th St',
        'distance': 330,
        'formattedAddress': ['335 Washington St (btw 3rd & 4th St)',
         'Hoboken, NJ 07030',
         'United States'],
        'labeledLatLngs': [{'label': 'display',
          'lat': 40.74111027118701,
          'lng': -74.02973403877716}],
        'lat': 40.74111027118701,
        'lng': -74.02973403877716,
        'postalCode': '07030',
        'state': 'NJ'},
       'name': 'Bangkok City',
       'photos': {'count': 0, 'groups': []}}},
     {'reasons': {'count': 0,
       'items': [{'reasonName': 'globalInteractionReason',
         'summary': 'This spot is popular',
         'type': 'general'}]},
      'referralId': 'e-0-4a9c695cf964a520153720e3-81',
      'venue': {'categories': [{'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/food/italian_',
          'suffix': '.png'},
         'id': '4bf58dd8d48988d110941735',
         'name': 'Italian Restaurant',
         'pluralName': 'Italian Restaurants',
         'primary': True,
         'shortName': 'Italian'}],
       'delivery': {'id': '1016090',
        'provider': {'icon': {'name': '/delivery_provider_grubhub_20180129.png',
          'prefix': 'https://fastly.4sqi.net/img/general/cap/',
          'sizes': [40, 50]},
         'name': 'grubhub'},
        'url': 'https://www.grubhub.com/restaurant/trattoria-saporito-328-washington-st-hoboken/1016090?affiliate=1131&utm_source=foursquare-affiliate-network&utm_medium=affiliate&utm_campaign=1131&utm_content=1016090'},
       'id': '4a9c695cf964a520153720e3',
       'location': {'address': '328 Washington St',
        'cc': 'US',
        'city': 'Hoboken',
        'country': 'United States',
        'crossStreet': 'at 4th St.',
        'distance': 335,
        'formattedAddress': ['328 Washington St (at 4th St.)',
         'Hoboken, NJ 07030',
         'United States'],
        'labeledLatLngs': [{'label': 'display',
          'lat': 40.74099560550887,
          'lng': -74.02982745407989}],
        'lat': 40.74099560550887,
        'lng': -74.02982745407989,
        'postalCode': '07030',
        'state': 'NJ'},
       'name': 'Trattoria Saporito',
       'photos': {'count': 0, 'groups': []}}},
     {'reasons': {'count': 0,
       'items': [{'reasonName': 'globalInteractionReason',
         'summary': 'This spot is popular',
         'type': 'general'}]},
      'referralId': 'e-0-567b01d8498e9310602c8d10-82',
      'venue': {'categories': [{'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/food/italian_',
          'suffix': '.png'},
         'id': '4bf58dd8d48988d110941735',
         'name': 'Italian Restaurant',
         'pluralName': 'Italian Restaurants',
         'primary': True,
         'shortName': 'Italian'}],
       'delivery': {'id': '339803',
        'provider': {'icon': {'name': '/delivery_provider_grubhub_20180129.png',
          'prefix': 'https://fastly.4sqi.net/img/general/cap/',
          'sizes': [40, 50]},
         'name': 'grubhub'},
        'url': 'https://www.grubhub.com/restaurant/frankie--avas-italian-eatery-208-washington-st-hoboken/339803?affiliate=1131&utm_source=foursquare-affiliate-network&utm_medium=affiliate&utm_campaign=1131&utm_content=339803'},
       'id': '567b01d8498e9310602c8d10',
       'location': {'address': '210 Washington St',
        'cc': 'US',
        'city': 'Hoboken',
        'country': 'United States',
        'distance': 492,
        'formattedAddress': ['210 Washington St',
         'Hoboken, NJ 07030',
         'United States'],
        'labeledLatLngs': [{'label': 'display',
          'lat': 40.739127253280024,
          'lng': -74.03044999672638}],
        'lat': 40.739127253280024,
        'lng': -74.03044999672638,
        'postalCode': '07030',
        'state': 'NJ'},
       'name': "Frankie & Ava's Italian Eatery",
       'photos': {'count': 0, 'groups': []}}},
     {'reasons': {'count': 0,
       'items': [{'reasonName': 'globalInteractionReason',
         'summary': 'This spot is popular',
         'type': 'general'}]},
      'referralId': 'e-0-4b44d83ef964a520f4fd25e3-83',
      'venue': {'categories': [{'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/food/pizza_',
          'suffix': '.png'},
         'id': '4bf58dd8d48988d1ca941735',
         'name': 'Pizza Place',
         'pluralName': 'Pizza Places',
         'primary': True,
         'shortName': 'Pizza'}],
       'delivery': {'id': '70913',
        'provider': {'icon': {'name': '/delivery_provider_grubhub_20180129.png',
          'prefix': 'https://fastly.4sqi.net/img/general/cap/',
          'sizes': [40, 50]},
         'name': 'grubhub'},
        'url': 'https://www.grubhub.com/restaurant/marios-classic-pizza-742-garden-st-hoboken/70913?affiliate=1131&utm_source=foursquare-affiliate-network&utm_medium=affiliate&utm_campaign=1131&utm_content=70913'},
       'id': '4b44d83ef964a520f4fd25e3',
       'location': {'address': '742 Garden St',
        'cc': 'US',
        'city': 'Hoboken',
        'country': 'United States',
        'crossStreet': '8th',
        'distance': 388,
        'formattedAddress': ['742 Garden St (8th)',
         'Hoboken, NJ 07030',
         'United States'],
        'labeledLatLngs': [{'label': 'display',
          'lat': 40.74636362311963,
          'lng': -74.03015162892913}],
        'lat': 40.74636362311963,
        'lng': -74.03015162892913,
        'postalCode': '07030',
        'state': 'NJ'},
       'name': "Mario's Classic Pizza Café",
       'photos': {'count': 0, 'groups': []}}},
     {'reasons': {'count': 0,
       'items': [{'reasonName': 'globalInteractionReason',
         'summary': 'This spot is popular',
         'type': 'general'}]},
      'referralId': 'e-0-578d640f498e64162d22e6e9-84',
      'venue': {'categories': [{'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/food/dessert_',
          'suffix': '.png'},
         'id': '4bf58dd8d48988d1d0941735',
         'name': 'Dessert Shop',
         'pluralName': 'Dessert Shops',
         'primary': True,
         'shortName': 'Desserts'}],
       'delivery': {'id': '531293',
        'provider': {'icon': {'name': '/delivery_provider_grubhub_20180129.png',
          'prefix': 'https://fastly.4sqi.net/img/general/cap/',
          'sizes': [40, 50]},
         'name': 'grubhub'},
        'url': 'https://www.grubhub.com/restaurant/juns-macaron-gelato-410-washington-st-hoboken/531293?affiliate=1131&utm_source=foursquare-affiliate-network&utm_medium=affiliate&utm_campaign=1131&utm_content=531293'},
       'id': '578d640f498e64162d22e6e9',
       'location': {'address': '410 Washington St',
        'cc': 'US',
        'city': 'Hoboken',
        'country': 'United States',
        'distance': 291,
        'formattedAddress': ['410 Washington St',
         'Hoboken, NJ 07030',
         'United States'],
        'labeledLatLngs': [{'label': 'display',
          'lat': 40.741696780507226,
          'lng': -74.02965104741823}],
        'lat': 40.741696780507226,
        'lng': -74.02965104741823,
        'postalCode': '07030',
        'state': 'NJ'},
       'name': "Jun's Macaron Gelato",
       'photos': {'count': 0, 'groups': []}}},
     {'reasons': {'count': 0,
       'items': [{'reasonName': 'globalInteractionReason',
         'summary': 'This spot is popular',
         'type': 'general'}]},
      'referralId': 'e-0-522cb84211d20c34ef82194f-85',
      'venue': {'categories': [{'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/nightlife/pub_',
          'suffix': '.png'},
         'id': '4bf58dd8d48988d11b941735',
         'name': 'Pub',
         'pluralName': 'Pubs',
         'primary': True,
         'shortName': 'Pub'}],
       'id': '522cb84211d20c34ef82194f',
       'location': {'address': '734 Willow Ave',
        'cc': 'US',
        'city': 'Hoboken',
        'country': 'United States',
        'crossStreet': 'between 7th and 8th st.',
        'distance': 354,
        'formattedAddress': ['734 Willow Ave (between 7th and 8th st.)',
         'Hoboken, NJ 07030',
         'United States'],
        'labeledLatLngs': [{'label': 'display',
          'lat': 40.746477358647326,
          'lng': -74.03203234301458}],
        'lat': 40.746477358647326,
        'lng': -74.03203234301458,
        'postalCode': '07030',
        'state': 'NJ'},
       'name': "Finnegan's Pub",
       'photos': {'count': 0, 'groups': []},
       'venuePage': {'id': '66978511'}}},
     {'reasons': {'count': 0,
       'items': [{'reasonName': 'globalInteractionReason',
         'summary': 'This spot is popular',
         'type': 'general'}]},
      'referralId': 'e-0-4bae42c4f964a520059a3be3-86',
      'venue': {'categories': [{'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/shops/food_liquor_',
          'suffix': '.png'},
         'id': '4bf58dd8d48988d186941735',
         'name': 'Liquor Store',
         'pluralName': 'Liquor Stores',
         'primary': True,
         'shortName': 'Liquor Store'}],
       'id': '4bae42c4f964a520059a3be3',
       'location': {'address': '700 Park Ave',
        'cc': 'US',
        'city': 'Hoboken',
        'country': 'United States',
        'distance': 228,
        'formattedAddress': ['700 Park Ave',
         'Hoboken, NJ 07030',
         'United States'],
        'labeledLatLngs': [{'label': 'display',
          'lat': 40.745235422392,
          'lng': -74.03144835634754}],
        'lat': 40.745235422392,
        'lng': -74.03144835634754,
        'postalCode': '07030',
        'state': 'NJ'},
       'name': 'Garden Wine & Liquor',
       'photos': {'count': 0, 'groups': []}}},
     {'reasons': {'count': 0,
       'items': [{'reasonName': 'globalInteractionReason',
         'summary': 'This spot is popular',
         'type': 'general'}]},
      'referralId': 'e-0-49cf90f2f964a520ae5a1fe3-87',
      'venue': {'categories': [{'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/nightlife/pub_',
          'suffix': '.png'},
         'id': '4bf58dd8d48988d116941735',
         'name': 'Bar',
         'pluralName': 'Bars',
         'primary': True,
         'shortName': 'Bar'}],
       'id': '49cf90f2f964a520ae5a1fe3',
       'location': {'address': '616 Grand St',
        'cc': 'US',
        'city': 'Hoboken',
        'country': 'United States',
        'crossStreet': 'btwn 6th & 7th Ave.',
        'distance': 270,
        'formattedAddress': ['616 Grand St (btwn 6th & 7th Ave.)',
         'Hoboken, NJ 07030',
         'United States'],
        'labeledLatLngs': [{'label': 'display',
          'lat': 40.745132558114484,
          'lng': -74.03449741670644}],
        'lat': 40.745132558114484,
        'lng': -74.03449741670644,
        'postalCode': '07030',
        'state': 'NJ'},
       'name': "Willie McBride's",
       'photos': {'count': 0, 'groups': []}}},
     {'reasons': {'count': 0,
       'items': [{'reasonName': 'globalInteractionReason',
         'summary': 'This spot is popular',
         'type': 'general'}]},
      'referralId': 'e-0-4b7f4011f964a5205f2230e3-88',
      'venue': {'categories': [{'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/shops/bookstore_',
          'suffix': '.png'},
         'id': '4bf58dd8d48988d114951735',
         'name': 'Bookstore',
         'pluralName': 'Bookstores',
         'primary': True,
         'shortName': 'Bookstore'}],
       'id': '4b7f4011f964a5205f2230e3',
       'location': {'address': '510 Washington St',
        'cc': 'US',
        'city': 'Hoboken',
        'country': 'United States',
        'crossStreet': '5th and Washington',
        'distance': 264,
        'formattedAddress': ['510 Washington St (5th and Washington)',
         'Hoboken, NJ 07030',
         'United States'],
        'labeledLatLngs': [{'label': 'display',
          'lat': 40.742582692129005,
          'lng': -74.02939078496166}],
        'lat': 40.742582692129005,
        'lng': -74.02939078496166,
        'postalCode': '07030',
        'state': 'NJ'},
       'name': 'Symposia Community Book Store',
       'photos': {'count': 0, 'groups': []}}},
     {'reasons': {'count': 0,
       'items': [{'reasonName': 'globalInteractionReason',
         'summary': 'This spot is popular',
         'type': 'general'}]},
      'referralId': 'e-0-49eabe84f964a52090661fe3-89',
      'venue': {'categories': [{'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/food/mexican_',
          'suffix': '.png'},
         'id': '4bf58dd8d48988d1c1941735',
         'name': 'Mexican Restaurant',
         'pluralName': 'Mexican Restaurants',
         'primary': True,
         'shortName': 'Mexican'}],
       'delivery': {'id': '70927',
        'provider': {'icon': {'name': '/delivery_provider_grubhub_20180129.png',
          'prefix': 'https://fastly.4sqi.net/img/general/cap/',
          'sizes': [40, 50]},
         'name': 'grubhub'},
        'url': 'https://www.grubhub.com/restaurant/qdoba-mexican-grill-hoboken-400-washington-st-hoboken/70927?affiliate=1131&utm_source=foursquare-affiliate-network&utm_medium=affiliate&utm_campaign=1131&utm_content=70927'},
       'id': '49eabe84f964a52090661fe3',
       'location': {'address': '400 Washington St',
        'cc': 'US',
        'city': 'Hoboken',
        'country': 'United States',
        'distance': 295,
        'formattedAddress': ['400 Washington St',
         'Hoboken, NJ 07030',
         'United States'],
        'labeledLatLngs': [{'label': 'display',
          'lat': 40.7414127,
          'lng': -74.0299258}],
        'lat': 40.7414127,
        'lng': -74.0299258,
        'postalCode': '07030',
        'state': 'NJ'},
       'name': 'QDOBA Mexican Eats',
       'photos': {'count': 0, 'groups': []}}},
     {'reasons': {'count': 0,
       'items': [{'reasonName': 'globalInteractionReason',
         'summary': 'This spot is popular',
         'type': 'general'}]},
      'referralId': 'e-0-4c13f99d7f7f2d7fda1ae068-90',
      'venue': {'categories': [{'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/arts_entertainment/musicvenue_',
          'suffix': '.png'},
         'id': '5032792091d4c4b30a586d5c',
         'name': 'Concert Hall',
         'pluralName': 'Concert Halls',
         'primary': True,
         'shortName': 'Concert Hall'}],
       'id': '4c13f99d7f7f2d7fda1ae068',
       'location': {'address': '500 Hudson St',
        'cc': 'US',
        'city': 'Hoboken',
        'country': 'United States',
        'distance': 405,
        'formattedAddress': ['500 Hudson St',
         'Hoboken, NJ 07030',
         'United States'],
        'labeledLatLngs': [{'label': 'display',
          'lat': 40.742485181523456,
          'lng': -74.02769242999422}],
        'lat': 40.742485181523456,
        'lng': -74.02769242999422,
        'postalCode': '07030',
        'state': 'NJ'},
       'name': 'DeBaun Center For Performing Arts',
       'photos': {'count': 0, 'groups': []}}},
     {'reasons': {'count': 0,
       'items': [{'reasonName': 'globalInteractionReason',
         'summary': 'This spot is popular',
         'type': 'general'}]},
      'referralId': 'e-0-4baa30e4f964a520e1513ae3-91',
      'venue': {'categories': [{'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/parks_outdoors/dogrun_',
          'suffix': '.png'},
         'id': '4bf58dd8d48988d1e5941735',
         'name': 'Dog Run',
         'pluralName': 'Dog Runs',
         'primary': True,
         'shortName': 'Dog Run'}],
       'id': '4baa30e4f964a520e1513ae3',
       'location': {'address': 'Hudson St',
        'cc': 'US',
        'city': 'Hoboken',
        'country': 'United States',
        'crossStreet': 'btwn 4th & 5th St',
        'distance': 402,
        'formattedAddress': ['Hudson St (btwn 4th & 5th St)',
         'Hoboken, NJ 07030',
         'United States'],
        'labeledLatLngs': [{'label': 'display',
          'lat': 40.7415352934731,
          'lng': -74.02821400762662}],
        'lat': 40.7415352934731,
        'lng': -74.02821400762662,
        'postalCode': '07030',
        'state': 'NJ'},
       'name': 'Stevens Park Dog Run',
       'photos': {'count': 0, 'groups': []}}},
     {'reasons': {'count': 0,
       'items': [{'reasonName': 'globalInteractionReason',
         'summary': 'This spot is popular',
         'type': 'general'}]},
      'referralId': 'e-0-4bf980e4b182c9b6290e795a-92',
      'venue': {'categories': [{'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/parks_outdoors/park_',
          'suffix': '.png'},
         'id': '4bf58dd8d48988d163941735',
         'name': 'Park',
         'pluralName': 'Parks',
         'primary': True,
         'shortName': 'Park'}],
       'id': '4bf980e4b182c9b6290e795a',
       'location': {'address': '401 Hudson Street',
        'cc': 'US',
        'city': 'Hoboken',
        'country': 'United States',
        'distance': 420,
        'formattedAddress': ['401 Hudson Street',
         'Hoboken, NJ 07030',
         'United States'],
        'labeledLatLngs': [{'label': 'display',
          'lat': 40.741414786438206,
          'lng': -74.02806427859284}],
        'lat': 40.741414786438206,
        'lng': -74.02806427859284,
        'postalCode': '07030',
        'state': 'NJ'},
       'name': 'Stevens Park',
       'photos': {'count': 0, 'groups': []}}},
     {'reasons': {'count': 0,
       'items': [{'reasonName': 'globalInteractionReason',
         'summary': 'This spot is popular',
         'type': 'general'}]},
      'referralId': 'e-0-4af5d847f964a520aefd21e3-93',
      'venue': {'categories': [{'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/food/pizza_',
          'suffix': '.png'},
         'id': '4bf58dd8d48988d1ca941735',
         'name': 'Pizza Place',
         'pluralName': 'Pizza Places',
         'primary': True,
         'shortName': 'Pizza'}],
       'delivery': {'id': '260855',
        'provider': {'icon': {'name': '/delivery_provider_grubhub_20180129.png',
          'prefix': 'https://fastly.4sqi.net/img/general/cap/',
          'sizes': [40, 50]},
         'name': 'grubhub'},
        'url': 'https://www.grubhub.com/restaurant/hs-giovannis-restaurant-and-pizzeria-603-washington-st-hoboken/260855?affiliate=1131&utm_source=foursquare-affiliate-network&utm_medium=affiliate&utm_campaign=1131&utm_content=260855'},
       'id': '4af5d847f964a520aefd21e3',
       'location': {'address': '603 Washington St',
        'cc': 'US',
        'city': 'Hoboken',
        'country': 'United States',
        'crossStreet': 'at 6th St.',
        'distance': 284,
        'formattedAddress': ['603 Washington St (at 6th St.)',
         'Hoboken, NJ 07030',
         'United States'],
        'labeledLatLngs': [{'label': 'display',
          'lat': 40.74350821413455,
          'lng': -74.02901687177996}],
        'lat': 40.74350821413455,
        'lng': -74.02901687177996,
        'postalCode': '07030',
        'state': 'NJ'},
       'name': "H&S Giovanni's",
       'photos': {'count': 0, 'groups': []}}},
     {'reasons': {'count': 0,
       'items': [{'reasonName': 'globalInteractionReason',
         'summary': 'This spot is popular',
         'type': 'general'}]},
      'referralId': 'e-0-4b804bcaf964a5204d6530e3-94',
      'venue': {'categories': [{'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/shops/videogames_',
          'suffix': '.png'},
         'id': '4bf58dd8d48988d10b951735',
         'name': 'Video Game Store',
         'pluralName': 'Video Game Stores',
         'primary': True,
         'shortName': 'Video Games'}],
       'id': '4b804bcaf964a5204d6530e3',
       'location': {'address': '408 Washington St',
        'cc': 'US',
        'city': 'Hoboken',
        'country': 'United States',
        'distance': 295,
        'formattedAddress': ['408 Washington St',
         'Hoboken, NJ 07030',
         'United States'],
        'labeledLatLngs': [{'label': 'display',
          'lat': 40.741620549066845,
          'lng': -74.02966563417021}],
        'lat': 40.741620549066845,
        'lng': -74.02966563417021,
        'postalCode': '07030',
        'state': 'NJ'},
       'name': 'GameStop',
       'photos': {'count': 0, 'groups': []}}},
     {'reasons': {'count': 0,
       'items': [{'reasonName': 'globalInteractionReason',
         'summary': 'This spot is popular',
         'type': 'general'}]},
      'referralId': 'e-0-5106f9db830220a8f4821b31-95',
      'venue': {'categories': [{'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/shops/pharmacy_',
          'suffix': '.png'},
         'id': '4bf58dd8d48988d10f951735',
         'name': 'Pharmacy',
         'pluralName': 'Pharmacies',
         'primary': True,
         'shortName': 'Pharmacy'}],
       'id': '5106f9db830220a8f4821b31',
       'location': {'address': '511 Washington St',
        'cc': 'US',
        'city': 'Hoboken',
        'country': 'United States',
        'crossStreet': '5th Street',
        'distance': 261,
        'formattedAddress': ['511 Washington St (5th Street)',
         'Hoboken, NJ 07030',
         'United States'],
        'labeledLatLngs': [{'label': 'display',
          'lat': 40.74285137270151,
          'lng': -74.02933359146117}],
        'lat': 40.74285137270151,
        'lng': -74.02933359146117,
        'postalCode': '07030',
        'state': 'NJ'},
       'name': 'Medicine Man Pharmacy',
       'photos': {'count': 0, 'groups': []},
       'venuePage': {'id': '46953460'}}},
     {'reasons': {'count': 0,
       'items': [{'reasonName': 'globalInteractionReason',
         'summary': 'This spot is popular',
         'type': 'general'}]},
      'referralId': 'e-0-4e7a5f94091ad40a876bbcda-96',
      'venue': {'categories': [{'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/shops/mobilephoneshop_',
          'suffix': '.png'},
         'id': '4f04afc02fb6e1c99f3db0bc',
         'name': 'Mobile Phone Shop',
         'pluralName': 'Mobile Phone Shops',
         'primary': True,
         'shortName': 'Mobile Phones'}],
       'id': '4e7a5f94091ad40a876bbcda',
       'location': {'address': '507 Washington St',
        'cc': 'US',
        'city': 'Hoboken',
        'country': 'United States',
        'distance': 299,
        'formattedAddress': ['507 Washington St',
         'Hoboken, NJ 07030',
         'United States'],
        'labeledLatLngs': [{'label': 'display',
          'lat': 40.742591254936954,
          'lng': -74.02894735336304}],
        'lat': 40.742591254936954,
        'lng': -74.02894735336304,
        'postalCode': '07030',
        'state': 'NJ'},
       'name': 'Verizon Wireless',
       'photos': {'count': 0, 'groups': []}}},
     {'reasons': {'count': 0,
       'items': [{'reasonName': 'globalInteractionReason',
         'summary': 'This spot is popular',
         'type': 'general'}]},
      'referralId': 'e-0-49e8d1a7f964a52097651fe3-97',
      'venue': {'categories': [{'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/nightlife/pub_',
          'suffix': '.png'},
         'id': '4bf58dd8d48988d116941735',
         'name': 'Bar',
         'pluralName': 'Bars',
         'primary': True,
         'shortName': 'Bar'}],
       'delivery': {'id': '447628',
        'provider': {'icon': {'name': '/delivery_provider_grubhub_20180129.png',
          'prefix': 'https://fastly.4sqi.net/img/general/cap/',
          'sizes': [40, 50]},
         'name': 'grubhub'},
        'url': 'https://www.grubhub.com/restaurant/farside-tavern-531-washington-st-hoboken/447628?affiliate=1131&utm_source=foursquare-affiliate-network&utm_medium=affiliate&utm_campaign=1131&utm_content=447628'},
       'id': '49e8d1a7f964a52097651fe3',
       'location': {'address': '531 Washington St',
        'cc': 'US',
        'city': 'Hoboken',
        'country': 'United States',
        'crossStreet': 'btw 5th & 6th on Washington',
        'distance': 272,
        'formattedAddress': ['531 Washington St (btw 5th & 6th on Washington)',
         'Hoboken, NJ 07030',
         'United States'],
        'labeledLatLngs': [{'label': 'display',
          'lat': 40.7431982448912,
          'lng': -74.02914504551752}],
        'lat': 40.7431982448912,
        'lng': -74.02914504551752,
        'postalCode': '07030',
        'state': 'NJ'},
       'name': 'Farside',
       'photos': {'count': 0, 'groups': []}}},
     {'reasons': {'count': 0,
       'items': [{'reasonName': 'globalInteractionReason',
         'summary': 'This spot is popular',
         'type': 'general'}]},
      'referralId': 'e-0-4af8f9a1f964a520bd1022e3-98',
      'venue': {'categories': [{'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/shops/pharmacy_',
          'suffix': '.png'},
         'id': '4bf58dd8d48988d10f951735',
         'name': 'Pharmacy',
         'pluralName': 'Pharmacies',
         'primary': True,
         'shortName': 'Pharmacy'}],
       'id': '4af8f9a1f964a520bd1022e3',
       'location': {'address': '811 Clinton St',
        'cc': 'US',
        'city': 'Hoboken',
        'country': 'United States',
        'crossStreet': '8th St.',
        'distance': 472,
        'formattedAddress': ['811 Clinton St (8th St.)',
         'Hoboken, NJ 07030',
         'United States'],
        'labeledLatLngs': [{'label': 'display',
          'lat': 40.747545945848806,
          'lng': -74.03249261837296}],
        'lat': 40.747545945848806,
        'lng': -74.03249261837296,
        'postalCode': '07030',
        'state': 'NJ'},
       'name': 'CVS pharmacy',
       'photos': {'count': 0, 'groups': []}}},
     {'reasons': {'count': 0,
       'items': [{'reasonName': 'globalInteractionReason',
         'summary': 'This spot is popular',
         'type': 'general'}]},
      'referralId': 'e-0-529fc55011d2e9632e382ae3-99',
      'venue': {'categories': [{'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/food/vietnamese_',
          'suffix': '.png'},
         'id': '4bf58dd8d48988d14a941735',
         'name': 'Vietnamese Restaurant',
         'pluralName': 'Vietnamese Restaurants',
         'primary': True,
         'shortName': 'Vietnamese'}],
       'delivery': {'id': '564066',
        'provider': {'icon': {'name': '/delivery_provider_grubhub_20180129.png',
          'prefix': 'https://fastly.4sqi.net/img/general/cap/',
          'sizes': [40, 50]},
         'name': 'grubhub'},
        'url': 'https://www.grubhub.com/restaurant/pho-nomenon-noodle-and-grill-516-washington-st-hoboken/564066?affiliate=1131&utm_source=foursquare-affiliate-network&utm_medium=affiliate&utm_campaign=1131&utm_content=564066'},
       'id': '529fc55011d2e9632e382ae3',
       'location': {'address': '516 Washington St',
        'cc': 'US',
        'city': 'Hoboken',
        'country': 'United States',
        'crossStreet': '5th Street',
        'distance': 267,
        'formattedAddress': ['516 Washington St (5th Street)',
         'Hoboken, NJ 07030',
         'United States'],
        'labeledLatLngs': [{'label': 'display',
          'lat': 40.74273338144599,
          'lng': -74.02929806418662}],
        'lat': 40.74273338144599,
        'lng': -74.02929806418662,
        'postalCode': '07030',
        'state': 'NJ'},
       'name': 'Pho Nomenon',
       'photos': {'count': 0, 'groups': []}}}],
    'name': 'recommended',
    'type': 'Recommended Places'}],
  'headerFullLocation': 'Hoboken',
  'headerLocation': 'Hoboken',
  'headerLocationGranularity': 'city',
  'suggestedBounds': {'ne': {'lat': 40.7478066045, 'lng': -74.0264467971303},
   'sw': {'lat': 40.738806595499995, 'lng': -74.0383036028697}},
  'suggestedFilters': {'filters': [{'key': 'openNow', 'name': 'Open now'},
    {'key': 'price', 'name': '$-$$$$'}],
   'header': 'Tap to show:'},
  'totalResults': 112}}
Foursquare Part 4
Now we start pulling the data from Foursquare into a dataframe so we can manipulate and use it.

In [6]:
def get_category_type(row):
    try:
        categories_list = row['categories']
    except:
        categories_list = row['venue.categories']
        
    if len(categories_list) == 0:
        return None
    else:
        return categories_list[0]['name']
In [7]:
#pull the actual data from the Foursquare API

venues = results['response']['groups'][0]['items']
nearby_venues = json_normalize(venues)
filtered_columns = ['venue.name', 'venue.id', 'venue.categories', 'venue.location.lat', 'venue.location.lng']
nearby_venues
nearby_venues =nearby_venues.loc[:, filtered_columns]

# filter the category for each row
nearby_venues['venue.categories'] = nearby_venues.apply(get_category_type, axis=1)

nearby_venues
Out[7]:
venue.name	venue.id	venue.categories	venue.location.lat	venue.location.lng
0	Work It Out-A Fitness Boutique	4cdf46dadb125481eb4236ce	Gym / Fitness Center	40.744356	-74.032567
1	Onieal's Restaurant & Bar	45e9482df964a52075431fe3	Pub	40.741608	-74.032304
2	Sweet	4a4e740ff964a5207bae1fe3	Bakery	40.741623	-74.031523
3	Church Square Park	49e9e49df964a5200a661fe3	Park	40.742152	-74.032230
4	Grand Vin	56d3b920498ec4e1c67c0907	Cocktail Bar	40.743209	-74.035099
5	Fiore's Deli	4a1098a2f964a520e3761fe3	Deli / Bodega	40.742995	-74.035981
6	O'Bagel	56daf06fcd107605ef3d86ea	Bagel Shop	40.743603	-74.029173
7	Ayame Hibachi & Sushi	4dbc9859f7b1ab37dd636d12	Sushi Restaurant	40.743105	-74.029213
8	Kung Fu Tea	57168865498e9517f09fa03d	Bubble Tea Shop	40.743375	-74.029450
9	Anthropologie	51f1c7e1498e7425c21efab6	Women's Store	40.741838	-74.029662
10	Karma Kafe	582dfc9565be5809f6a964ed	Indian Restaurant	40.742373	-74.029376
11	Dom's Bakery	4b660130f964a520ce0d2be3	Bakery	40.743387	-74.034836
12	Fleet Feet Sports	4b784457f964a520cdc02ee3	Sporting Goods Shop	40.743646	-74.029078
13	Court Street Bar & Restaurant	4a7eff1cf964a5206ff21fe3	American Restaurant	40.743322	-74.028615
14	Hudson River Athletics	51c892e02fc6ae7a5278a97e	Gym / Fitness Center	40.745563	-74.032841
15	Empire Coffee & Tea	49f37b88f964a520a26a1fe3	Coffee Shop	40.741375	-74.030515
16	Mamoun's Falafel	4d9368407b5ea1437d14c8b8	Falafel Restaurant	40.742303	-74.029465
17	Zack's Oak Bar & Restaurant	49f26862f964a520296a1fe3	American Restaurant	40.740640	-74.033826
18	Cafe Michelina	4a7b5b6bf964a520c8ea1fe3	Italian Restaurant	40.742278	-74.030218
19	Grimaldi's	4ca50f407334236a60ef1258	Pizza Place	40.741674	-74.029578
20	Robongi	4cdb36c1958f236a15a7ab03	Sushi Restaurant	40.742879	-74.029280
21	Old German Bakery	4d3b7bdd687ca35d1e8a94c4	Bakery	40.741147	-74.029810
22	Bareburger	53ed3b37498e4151087521a9	Burger Joint	40.742694	-74.029070
23	Cucharamama	49e2a407f964a52045621fe3	South American Restaurant	40.740807	-74.034582
24	Cork City Pub	4d4218cd607b6dcb31df08c6	Pub	40.740105	-74.030868
25	Ben & Jerry's	49dfb562f964a52001611fe3	Ice Cream Shop	40.741430	-74.029484
26	Church Square Dog Park	4b93cf35f964a520c65234e3	Dog Run	40.742203	-74.033038
27	The Cuban	4eb1b6859adfb95b77765bf9	Cuban Restaurant	40.741012	-74.029732
28	Aether Game Cafe	57127a9d498e648da026a585	Gaming Cafe	40.742813	-74.029184
29	Benny Tudino's	4a9ac1b1f964a520813220e3	Pizza Place	40.744198	-74.028896
30	Warby Parker	5bede1881953f3002c8bd30e	Optical Shop	40.743458	-74.029408
31	Surya Yoga Academy	4b5cd37cf964a520f04529e3	Yoga Studio	40.744102	-74.029007
32	Hoboken Pet	4eb039c4e5fa708105c903ac	Pet Store	40.743035	-74.029244
33	Moran's Pub	4ad12c5ef964a5203ddd20e3	Bar	40.742553	-74.031192
34	Zafra	4a453f9cf964a520f4a71fe3	Cuban Restaurant	40.740659	-74.033802
35	Tutta Pesca	57f83f7acd10164c2ec1956f	Seafood Restaurant	40.740163	-74.031284
36	Ultramarinos	4c03f2fe39d476b0f5c530a7	Latin American Restaurant	40.740620	-74.033490
37	Dozzino	4c60c4a1de6920a111ed9664	Pizza Place	40.744612	-74.035632
38	Otto Strada	527f3d1711d2f7f001c656b2	Italian Restaurant	40.746604	-74.031161
39	Illuzion	4a9578dff964a520562320e3	Japanese Restaurant	40.741229	-74.029582
40	Bluestone Lane	58e7ed715f67173549fe6246	Coffee Shop	40.741601	-74.029410
41	Giovanni D'Italia Shoe Repair	4b7415e2f964a52023c72de3	Shoe Repair	40.745382	-74.032227
42	Hoboken Burrito	49ee57f6f964a5204f681fe3	Mexican Restaurant	40.741822	-74.031881
43	Townhouse No 620	4bddbf6be75c0f47f171c503	Boutique	40.744128	-74.029070
44	Jefferson’s Coffee	5a888bc2c47cf91545daec08	Coffee Shop	40.743368	-74.029135
45	Vito's Italian Deli	4a8da189f964a520501020e3	Sandwich Place	40.746401	-74.028310
46	Makai Poke Co	58c470fd37da1d593431c33a	Poke Place	40.742852	-74.029213
47	Chicken Factory	4c41e1aee26920a1981e5fe7	Korean Restaurant	40.742975	-74.029083
48	aaRaa	4c682b63e1da1b8d4a729fc3	Jewelry Store	40.744487	-74.035039
49	Midtown Philly Steaks	51070e83e4b0d92d935e04a3	Sandwich Place	40.742983	-74.029211
50	Mamoun’s Falafel	5a6b6047f427de038c51031c	Falafel Restaurant	40.740090	-74.030374
51	Mr Wrap's	4ad89c0bf964a520d31221e3	American Restaurant	40.746308	-74.030003
52	Tunes	4a74aee1f964a5202ddf1fe3	Record Shop	40.739426	-74.030096
53	Margherita's	4a775512f964a5202ee41fe3	Italian Restaurant	40.746034	-74.028412
54	Mikie Squared Bar & Grill	4a3ad481f964a52057a01fe3	Sports Bar	40.744032	-74.028956
55	Total Nutrition Kitchen	56cb57c7cd10aa9184195eec	Juice Bar	40.740892	-74.029896
56	Modern Nails and Spa	4c32368b3896e21eba28e890	Cosmetics Shop	40.745624	-74.028501
57	SEPHORA	5abbff66123a1973728506b7	Cosmetics Shop	40.740585	-74.029715
58	Maroon	4a9703def964a520fb2720e3	Tea Room	40.745307	-74.032359
59	Chipotle Mexican Grill	5065c556e4b0a44a76b324ce	Mexican Restaurant	40.739599	-74.029981
60	East LA	49e37784f964a52083621fe3	Mexican Restaurant	40.742509	-74.029496
61	The Stewed Cow	5131669e582f0b9471213351	Bar	40.742461	-74.036385
62	Makeover II	4bc3bdc2920eb71334f31d2c	Cosmetics Shop	40.740078	-74.030161
63	eMazzanti Technologies	4c928177418ea1cd8c86a585	Business Service	40.745843	-74.033968
64	Flatbread Grill	55d52901498ea18f871d5f9e	Restaurant	40.742767	-74.029281
65	Dunkin' Donuts	4b191a58f964a520ffd723e3	Donut Shop	40.744813	-74.028812
66	Pita Pit	56f136ad498ecc5661aa49ce	Sandwich Place	40.745729	-74.028582
67	Beowoof	4b79c7d0f964a520a1112fe3	Pet Store	40.742272	-74.029896
68	Bloom Spa And Nails	4bc1df1d2a89ef3bc1cdf288	Salon / Barbershop	40.741393	-74.029723
69	Caporrino's News & Liquor	4e4d8d3eae608af796ea0e86	Liquor Store	40.743922	-74.036102
70	Sauced	5ac2cec0419a9e0f679671f3	Sandwich Place	40.739274	-74.030205
71	Basic Food	4b035ae1f964a520e24e22e3	Grocery Store	40.739068	-74.030388
72	Cozy Cuts Pets Grooming, LLC	4e3d3837b0fb875af85e8b72	Pet Store	40.743622	-74.037186
73	Hudson Paperie	562c00b2498eb20825dcc6c0	Stationery Store	40.740540	-74.030045
74	Sri Thai	49e27ed2f964a5201c621fe3	Thai Restaurant	40.740103	-74.030955
75	Napoli’s Pizzeria	59fd168ec21cb1401894e47e	Pizza Place	40.739404	-74.034909
76	Hoboken Hot Bagels	4b76e28df964a52021672ee3	Bagel Shop	40.744448	-74.028822
77	Renaissance Pilates	4b800a9ef964a520144d30e3	Pilates Studio	40.740460	-74.027858
78	Hidden Grounds Coffee	5b5b1e0bd4cc9800394e3256	Coffee Shop	40.745117	-74.030493
79	Louise & Jerry's	4283ee00f964a52091221fe3	Dive Bar	40.740880	-74.029480
80	Bangkok City	4acce0aaf964a520d8c920e3	Thai Restaurant	40.741110	-74.029734
81	Trattoria Saporito	4a9c695cf964a520153720e3	Italian Restaurant	40.740996	-74.029827
82	Frankie & Ava's Italian Eatery	567b01d8498e9310602c8d10	Italian Restaurant	40.739127	-74.030450
83	Mario's Classic Pizza Café	4b44d83ef964a520f4fd25e3	Pizza Place	40.746364	-74.030152
84	Jun's Macaron Gelato	578d640f498e64162d22e6e9	Dessert Shop	40.741697	-74.029651
85	Finnegan's Pub	522cb84211d20c34ef82194f	Pub	40.746477	-74.032032
86	Garden Wine & Liquor	4bae42c4f964a520059a3be3	Liquor Store	40.745235	-74.031448
87	Willie McBride's	49cf90f2f964a520ae5a1fe3	Bar	40.745133	-74.034497
88	Symposia Community Book Store	4b7f4011f964a5205f2230e3	Bookstore	40.742583	-74.029391
89	QDOBA Mexican Eats	49eabe84f964a52090661fe3	Mexican Restaurant	40.741413	-74.029926
90	DeBaun Center For Performing Arts	4c13f99d7f7f2d7fda1ae068	Concert Hall	40.742485	-74.027692
91	Stevens Park Dog Run	4baa30e4f964a520e1513ae3	Dog Run	40.741535	-74.028214
92	Stevens Park	4bf980e4b182c9b6290e795a	Park	40.741415	-74.028064
93	H&S Giovanni's	4af5d847f964a520aefd21e3	Pizza Place	40.743508	-74.029017
94	GameStop	4b804bcaf964a5204d6530e3	Video Game Store	40.741621	-74.029666
95	Medicine Man Pharmacy	5106f9db830220a8f4821b31	Pharmacy	40.742851	-74.029334
96	Verizon Wireless	4e7a5f94091ad40a876bbcda	Mobile Phone Shop	40.742591	-74.028947
97	Farside	49e8d1a7f964a52097651fe3	Bar	40.743198	-74.029145
98	CVS pharmacy	4af8f9a1f964a520bd1022e3	Pharmacy	40.747546	-74.032493
99	Pho Nomenon	529fc55011d2e9632e382ae3	Vietnamese Restaurant	40.742733	-74.029298
In [8]:
#fix the column names so they look relatively normal

nearby_venues.columns = [col.split(".")[-1] for col in nearby_venues.columns]

nearby_venues
Out[8]:
name	id	categories	lat	lng
0	Work It Out-A Fitness Boutique	4cdf46dadb125481eb4236ce	Gym / Fitness Center	40.744356	-74.032567
1	Onieal's Restaurant & Bar	45e9482df964a52075431fe3	Pub	40.741608	-74.032304
2	Sweet	4a4e740ff964a5207bae1fe3	Bakery	40.741623	-74.031523
3	Church Square Park	49e9e49df964a5200a661fe3	Park	40.742152	-74.032230
4	Grand Vin	56d3b920498ec4e1c67c0907	Cocktail Bar	40.743209	-74.035099
5	Fiore's Deli	4a1098a2f964a520e3761fe3	Deli / Bodega	40.742995	-74.035981
6	O'Bagel	56daf06fcd107605ef3d86ea	Bagel Shop	40.743603	-74.029173
7	Ayame Hibachi & Sushi	4dbc9859f7b1ab37dd636d12	Sushi Restaurant	40.743105	-74.029213
8	Kung Fu Tea	57168865498e9517f09fa03d	Bubble Tea Shop	40.743375	-74.029450
9	Anthropologie	51f1c7e1498e7425c21efab6	Women's Store	40.741838	-74.029662
10	Karma Kafe	582dfc9565be5809f6a964ed	Indian Restaurant	40.742373	-74.029376
11	Dom's Bakery	4b660130f964a520ce0d2be3	Bakery	40.743387	-74.034836
12	Fleet Feet Sports	4b784457f964a520cdc02ee3	Sporting Goods Shop	40.743646	-74.029078
13	Court Street Bar & Restaurant	4a7eff1cf964a5206ff21fe3	American Restaurant	40.743322	-74.028615
14	Hudson River Athletics	51c892e02fc6ae7a5278a97e	Gym / Fitness Center	40.745563	-74.032841
15	Empire Coffee & Tea	49f37b88f964a520a26a1fe3	Coffee Shop	40.741375	-74.030515
16	Mamoun's Falafel	4d9368407b5ea1437d14c8b8	Falafel Restaurant	40.742303	-74.029465
17	Zack's Oak Bar & Restaurant	49f26862f964a520296a1fe3	American Restaurant	40.740640	-74.033826
18	Cafe Michelina	4a7b5b6bf964a520c8ea1fe3	Italian Restaurant	40.742278	-74.030218
19	Grimaldi's	4ca50f407334236a60ef1258	Pizza Place	40.741674	-74.029578
20	Robongi	4cdb36c1958f236a15a7ab03	Sushi Restaurant	40.742879	-74.029280
21	Old German Bakery	4d3b7bdd687ca35d1e8a94c4	Bakery	40.741147	-74.029810
22	Bareburger	53ed3b37498e4151087521a9	Burger Joint	40.742694	-74.029070
23	Cucharamama	49e2a407f964a52045621fe3	South American Restaurant	40.740807	-74.034582
24	Cork City Pub	4d4218cd607b6dcb31df08c6	Pub	40.740105	-74.030868
25	Ben & Jerry's	49dfb562f964a52001611fe3	Ice Cream Shop	40.741430	-74.029484
26	Church Square Dog Park	4b93cf35f964a520c65234e3	Dog Run	40.742203	-74.033038
27	The Cuban	4eb1b6859adfb95b77765bf9	Cuban Restaurant	40.741012	-74.029732
28	Aether Game Cafe	57127a9d498e648da026a585	Gaming Cafe	40.742813	-74.029184
29	Benny Tudino's	4a9ac1b1f964a520813220e3	Pizza Place	40.744198	-74.028896
30	Warby Parker	5bede1881953f3002c8bd30e	Optical Shop	40.743458	-74.029408
31	Surya Yoga Academy	4b5cd37cf964a520f04529e3	Yoga Studio	40.744102	-74.029007
32	Hoboken Pet	4eb039c4e5fa708105c903ac	Pet Store	40.743035	-74.029244
33	Moran's Pub	4ad12c5ef964a5203ddd20e3	Bar	40.742553	-74.031192
34	Zafra	4a453f9cf964a520f4a71fe3	Cuban Restaurant	40.740659	-74.033802
35	Tutta Pesca	57f83f7acd10164c2ec1956f	Seafood Restaurant	40.740163	-74.031284
36	Ultramarinos	4c03f2fe39d476b0f5c530a7	Latin American Restaurant	40.740620	-74.033490
37	Dozzino	4c60c4a1de6920a111ed9664	Pizza Place	40.744612	-74.035632
38	Otto Strada	527f3d1711d2f7f001c656b2	Italian Restaurant	40.746604	-74.031161
39	Illuzion	4a9578dff964a520562320e3	Japanese Restaurant	40.741229	-74.029582
40	Bluestone Lane	58e7ed715f67173549fe6246	Coffee Shop	40.741601	-74.029410
41	Giovanni D'Italia Shoe Repair	4b7415e2f964a52023c72de3	Shoe Repair	40.745382	-74.032227
42	Hoboken Burrito	49ee57f6f964a5204f681fe3	Mexican Restaurant	40.741822	-74.031881
43	Townhouse No 620	4bddbf6be75c0f47f171c503	Boutique	40.744128	-74.029070
44	Jefferson’s Coffee	5a888bc2c47cf91545daec08	Coffee Shop	40.743368	-74.029135
45	Vito's Italian Deli	4a8da189f964a520501020e3	Sandwich Place	40.746401	-74.028310
46	Makai Poke Co	58c470fd37da1d593431c33a	Poke Place	40.742852	-74.029213
47	Chicken Factory	4c41e1aee26920a1981e5fe7	Korean Restaurant	40.742975	-74.029083
48	aaRaa	4c682b63e1da1b8d4a729fc3	Jewelry Store	40.744487	-74.035039
49	Midtown Philly Steaks	51070e83e4b0d92d935e04a3	Sandwich Place	40.742983	-74.029211
50	Mamoun’s Falafel	5a6b6047f427de038c51031c	Falafel Restaurant	40.740090	-74.030374
51	Mr Wrap's	4ad89c0bf964a520d31221e3	American Restaurant	40.746308	-74.030003
52	Tunes	4a74aee1f964a5202ddf1fe3	Record Shop	40.739426	-74.030096
53	Margherita's	4a775512f964a5202ee41fe3	Italian Restaurant	40.746034	-74.028412
54	Mikie Squared Bar & Grill	4a3ad481f964a52057a01fe3	Sports Bar	40.744032	-74.028956
55	Total Nutrition Kitchen	56cb57c7cd10aa9184195eec	Juice Bar	40.740892	-74.029896
56	Modern Nails and Spa	4c32368b3896e21eba28e890	Cosmetics Shop	40.745624	-74.028501
57	SEPHORA	5abbff66123a1973728506b7	Cosmetics Shop	40.740585	-74.029715
58	Maroon	4a9703def964a520fb2720e3	Tea Room	40.745307	-74.032359
59	Chipotle Mexican Grill	5065c556e4b0a44a76b324ce	Mexican Restaurant	40.739599	-74.029981
60	East LA	49e37784f964a52083621fe3	Mexican Restaurant	40.742509	-74.029496
61	The Stewed Cow	5131669e582f0b9471213351	Bar	40.742461	-74.036385
62	Makeover II	4bc3bdc2920eb71334f31d2c	Cosmetics Shop	40.740078	-74.030161
63	eMazzanti Technologies	4c928177418ea1cd8c86a585	Business Service	40.745843	-74.033968
64	Flatbread Grill	55d52901498ea18f871d5f9e	Restaurant	40.742767	-74.029281
65	Dunkin' Donuts	4b191a58f964a520ffd723e3	Donut Shop	40.744813	-74.028812
66	Pita Pit	56f136ad498ecc5661aa49ce	Sandwich Place	40.745729	-74.028582
67	Beowoof	4b79c7d0f964a520a1112fe3	Pet Store	40.742272	-74.029896
68	Bloom Spa And Nails	4bc1df1d2a89ef3bc1cdf288	Salon / Barbershop	40.741393	-74.029723
69	Caporrino's News & Liquor	4e4d8d3eae608af796ea0e86	Liquor Store	40.743922	-74.036102
70	Sauced	5ac2cec0419a9e0f679671f3	Sandwich Place	40.739274	-74.030205
71	Basic Food	4b035ae1f964a520e24e22e3	Grocery Store	40.739068	-74.030388
72	Cozy Cuts Pets Grooming, LLC	4e3d3837b0fb875af85e8b72	Pet Store	40.743622	-74.037186
73	Hudson Paperie	562c00b2498eb20825dcc6c0	Stationery Store	40.740540	-74.030045
74	Sri Thai	49e27ed2f964a5201c621fe3	Thai Restaurant	40.740103	-74.030955
75	Napoli’s Pizzeria	59fd168ec21cb1401894e47e	Pizza Place	40.739404	-74.034909
76	Hoboken Hot Bagels	4b76e28df964a52021672ee3	Bagel Shop	40.744448	-74.028822
77	Renaissance Pilates	4b800a9ef964a520144d30e3	Pilates Studio	40.740460	-74.027858
78	Hidden Grounds Coffee	5b5b1e0bd4cc9800394e3256	Coffee Shop	40.745117	-74.030493
79	Louise & Jerry's	4283ee00f964a52091221fe3	Dive Bar	40.740880	-74.029480
80	Bangkok City	4acce0aaf964a520d8c920e3	Thai Restaurant	40.741110	-74.029734
81	Trattoria Saporito	4a9c695cf964a520153720e3	Italian Restaurant	40.740996	-74.029827
82	Frankie & Ava's Italian Eatery	567b01d8498e9310602c8d10	Italian Restaurant	40.739127	-74.030450
83	Mario's Classic Pizza Café	4b44d83ef964a520f4fd25e3	Pizza Place	40.746364	-74.030152
84	Jun's Macaron Gelato	578d640f498e64162d22e6e9	Dessert Shop	40.741697	-74.029651
85	Finnegan's Pub	522cb84211d20c34ef82194f	Pub	40.746477	-74.032032
86	Garden Wine & Liquor	4bae42c4f964a520059a3be3	Liquor Store	40.745235	-74.031448
87	Willie McBride's	49cf90f2f964a520ae5a1fe3	Bar	40.745133	-74.034497
88	Symposia Community Book Store	4b7f4011f964a5205f2230e3	Bookstore	40.742583	-74.029391
89	QDOBA Mexican Eats	49eabe84f964a52090661fe3	Mexican Restaurant	40.741413	-74.029926
90	DeBaun Center For Performing Arts	4c13f99d7f7f2d7fda1ae068	Concert Hall	40.742485	-74.027692
91	Stevens Park Dog Run	4baa30e4f964a520e1513ae3	Dog Run	40.741535	-74.028214
92	Stevens Park	4bf980e4b182c9b6290e795a	Park	40.741415	-74.028064
93	H&S Giovanni's	4af5d847f964a520aefd21e3	Pizza Place	40.743508	-74.029017
94	GameStop	4b804bcaf964a5204d6530e3	Video Game Store	40.741621	-74.029666
95	Medicine Man Pharmacy	5106f9db830220a8f4821b31	Pharmacy	40.742851	-74.029334
96	Verizon Wireless	4e7a5f94091ad40a876bbcda	Mobile Phone Shop	40.742591	-74.028947
97	Farside	49e8d1a7f964a52097651fe3	Bar	40.743198	-74.029145
98	CVS pharmacy	4af8f9a1f964a520bd1022e3	Pharmacy	40.747546	-74.032493
99	Pho Nomenon	529fc55011d2e9632e382ae3	Vietnamese Restaurant	40.742733	-74.029298
In [9]:
# find a list of unique categories from the API so we can see what may or may not fit for restaurants

nearby_venues['categories'].unique()
Out[9]:
array(['Gym / Fitness Center', 'Pub', 'Bakery', 'Park', 'Cocktail Bar',
       'Deli / Bodega', 'Bagel Shop', 'Sushi Restaurant',
       'Bubble Tea Shop', "Women's Store", 'Indian Restaurant',
       'Sporting Goods Shop', 'American Restaurant', 'Coffee Shop',
       'Falafel Restaurant', 'Italian Restaurant', 'Pizza Place',
       'Burger Joint', 'South American Restaurant', 'Ice Cream Shop',
       'Dog Run', 'Cuban Restaurant', 'Gaming Cafe', 'Optical Shop',
       'Yoga Studio', 'Pet Store', 'Bar', 'Seafood Restaurant',
       'Latin American Restaurant', 'Japanese Restaurant', 'Shoe Repair',
       'Mexican Restaurant', 'Boutique', 'Sandwich Place', 'Poke Place',
       'Korean Restaurant', 'Jewelry Store', 'Record Shop', 'Sports Bar',
       'Juice Bar', 'Cosmetics Shop', 'Tea Room', 'Business Service',
       'Restaurant', 'Donut Shop', 'Salon / Barbershop', 'Liquor Store',
       'Grocery Store', 'Stationery Store', 'Thai Restaurant',
       'Pilates Studio', 'Dive Bar', 'Dessert Shop', 'Bookstore',
       'Concert Hall', 'Video Game Store', 'Pharmacy', 'Mobile Phone Shop',
       'Vietnamese Restaurant'], dtype=object)
In [10]:
# creating a list of categorie to remove from our dataframe because they are not restaurants
# I am sure there is a function that can be written to do this at scale but since it was a small list, I did it manually

removal_list = ['Gym / Fitness Center', 'Bakery', 'Park', "Women's Store", 'Sporting Goods Shop', 'Dog Run', 'Gaming Cafe',
               'Optical Shop', 'Yoga Studio', 'Pet Store', 'Shoe Repair', 'Jewelry Store', 'Record Shop', 'Juice Bar', 
               'Cosmetics Shop', 'Business Service', 'Salon / Barbershop', 'Liquor Store', 'Grocery Store', 'Stationery Store',
               'Pilates Studio', 'Dessert Shop', 'Bookstore', 'Concert Hall', 'Video Game Store', 'Pharmacy', 'Mobile Phone Shop',
               'Deli / Bodega']

nearby_venues2 = nearby_venues.copy()


#getting a clear dataframe of just restaurants
nearby_venues2 = nearby_venues2[~nearby_venues2['categories'].isin(removal_list)]
nearby_venues2
Out[10]:
name	id	categories	lat	lng
1	Onieal's Restaurant & Bar	45e9482df964a52075431fe3	Pub	40.741608	-74.032304
4	Grand Vin	56d3b920498ec4e1c67c0907	Cocktail Bar	40.743209	-74.035099
6	O'Bagel	56daf06fcd107605ef3d86ea	Bagel Shop	40.743603	-74.029173
7	Ayame Hibachi & Sushi	4dbc9859f7b1ab37dd636d12	Sushi Restaurant	40.743105	-74.029213
8	Kung Fu Tea	57168865498e9517f09fa03d	Bubble Tea Shop	40.743375	-74.029450
10	Karma Kafe	582dfc9565be5809f6a964ed	Indian Restaurant	40.742373	-74.029376
13	Court Street Bar & Restaurant	4a7eff1cf964a5206ff21fe3	American Restaurant	40.743322	-74.028615
15	Empire Coffee & Tea	49f37b88f964a520a26a1fe3	Coffee Shop	40.741375	-74.030515
16	Mamoun's Falafel	4d9368407b5ea1437d14c8b8	Falafel Restaurant	40.742303	-74.029465
17	Zack's Oak Bar & Restaurant	49f26862f964a520296a1fe3	American Restaurant	40.740640	-74.033826
18	Cafe Michelina	4a7b5b6bf964a520c8ea1fe3	Italian Restaurant	40.742278	-74.030218
19	Grimaldi's	4ca50f407334236a60ef1258	Pizza Place	40.741674	-74.029578
20	Robongi	4cdb36c1958f236a15a7ab03	Sushi Restaurant	40.742879	-74.029280
22	Bareburger	53ed3b37498e4151087521a9	Burger Joint	40.742694	-74.029070
23	Cucharamama	49e2a407f964a52045621fe3	South American Restaurant	40.740807	-74.034582
24	Cork City Pub	4d4218cd607b6dcb31df08c6	Pub	40.740105	-74.030868
25	Ben & Jerry's	49dfb562f964a52001611fe3	Ice Cream Shop	40.741430	-74.029484
27	The Cuban	4eb1b6859adfb95b77765bf9	Cuban Restaurant	40.741012	-74.029732
29	Benny Tudino's	4a9ac1b1f964a520813220e3	Pizza Place	40.744198	-74.028896
33	Moran's Pub	4ad12c5ef964a5203ddd20e3	Bar	40.742553	-74.031192
34	Zafra	4a453f9cf964a520f4a71fe3	Cuban Restaurant	40.740659	-74.033802
35	Tutta Pesca	57f83f7acd10164c2ec1956f	Seafood Restaurant	40.740163	-74.031284
36	Ultramarinos	4c03f2fe39d476b0f5c530a7	Latin American Restaurant	40.740620	-74.033490
37	Dozzino	4c60c4a1de6920a111ed9664	Pizza Place	40.744612	-74.035632
38	Otto Strada	527f3d1711d2f7f001c656b2	Italian Restaurant	40.746604	-74.031161
39	Illuzion	4a9578dff964a520562320e3	Japanese Restaurant	40.741229	-74.029582
40	Bluestone Lane	58e7ed715f67173549fe6246	Coffee Shop	40.741601	-74.029410
42	Hoboken Burrito	49ee57f6f964a5204f681fe3	Mexican Restaurant	40.741822	-74.031881
43	Townhouse No 620	4bddbf6be75c0f47f171c503	Boutique	40.744128	-74.029070
44	Jefferson’s Coffee	5a888bc2c47cf91545daec08	Coffee Shop	40.743368	-74.029135
45	Vito's Italian Deli	4a8da189f964a520501020e3	Sandwich Place	40.746401	-74.028310
46	Makai Poke Co	58c470fd37da1d593431c33a	Poke Place	40.742852	-74.029213
47	Chicken Factory	4c41e1aee26920a1981e5fe7	Korean Restaurant	40.742975	-74.029083
49	Midtown Philly Steaks	51070e83e4b0d92d935e04a3	Sandwich Place	40.742983	-74.029211
50	Mamoun’s Falafel	5a6b6047f427de038c51031c	Falafel Restaurant	40.740090	-74.030374
51	Mr Wrap's	4ad89c0bf964a520d31221e3	American Restaurant	40.746308	-74.030003
53	Margherita's	4a775512f964a5202ee41fe3	Italian Restaurant	40.746034	-74.028412
54	Mikie Squared Bar & Grill	4a3ad481f964a52057a01fe3	Sports Bar	40.744032	-74.028956
58	Maroon	4a9703def964a520fb2720e3	Tea Room	40.745307	-74.032359
59	Chipotle Mexican Grill	5065c556e4b0a44a76b324ce	Mexican Restaurant	40.739599	-74.029981
60	East LA	49e37784f964a52083621fe3	Mexican Restaurant	40.742509	-74.029496
61	The Stewed Cow	5131669e582f0b9471213351	Bar	40.742461	-74.036385
64	Flatbread Grill	55d52901498ea18f871d5f9e	Restaurant	40.742767	-74.029281
65	Dunkin' Donuts	4b191a58f964a520ffd723e3	Donut Shop	40.744813	-74.028812
66	Pita Pit	56f136ad498ecc5661aa49ce	Sandwich Place	40.745729	-74.028582
70	Sauced	5ac2cec0419a9e0f679671f3	Sandwich Place	40.739274	-74.030205
74	Sri Thai	49e27ed2f964a5201c621fe3	Thai Restaurant	40.740103	-74.030955
75	Napoli’s Pizzeria	59fd168ec21cb1401894e47e	Pizza Place	40.739404	-74.034909
76	Hoboken Hot Bagels	4b76e28df964a52021672ee3	Bagel Shop	40.744448	-74.028822
78	Hidden Grounds Coffee	5b5b1e0bd4cc9800394e3256	Coffee Shop	40.745117	-74.030493
79	Louise & Jerry's	4283ee00f964a52091221fe3	Dive Bar	40.740880	-74.029480
80	Bangkok City	4acce0aaf964a520d8c920e3	Thai Restaurant	40.741110	-74.029734
81	Trattoria Saporito	4a9c695cf964a520153720e3	Italian Restaurant	40.740996	-74.029827
82	Frankie & Ava's Italian Eatery	567b01d8498e9310602c8d10	Italian Restaurant	40.739127	-74.030450
83	Mario's Classic Pizza Café	4b44d83ef964a520f4fd25e3	Pizza Place	40.746364	-74.030152
85	Finnegan's Pub	522cb84211d20c34ef82194f	Pub	40.746477	-74.032032
87	Willie McBride's	49cf90f2f964a520ae5a1fe3	Bar	40.745133	-74.034497
89	QDOBA Mexican Eats	49eabe84f964a52090661fe3	Mexican Restaurant	40.741413	-74.029926
93	H&S Giovanni's	4af5d847f964a520aefd21e3	Pizza Place	40.743508	-74.029017
97	Farside	49e8d1a7f964a52097651fe3	Bar	40.743198	-74.029145
99	Pho Nomenon	529fc55011d2e9632e382ae3	Vietnamese Restaurant	40.742733	-74.029298
Foursquare Part 5
Now let's get a list of venue ids so we can pull likes and add to our dataframe.

In [11]:
#let's get a list of venues

venue_id_list = nearby_venues2['id'].tolist()
venue_id_list
Out[11]:
['45e9482df964a52075431fe3',
 '56d3b920498ec4e1c67c0907',
 '56daf06fcd107605ef3d86ea',
 '4dbc9859f7b1ab37dd636d12',
 '57168865498e9517f09fa03d',
 '582dfc9565be5809f6a964ed',
 '4a7eff1cf964a5206ff21fe3',
 '49f37b88f964a520a26a1fe3',
 '4d9368407b5ea1437d14c8b8',
 '49f26862f964a520296a1fe3',
 '4a7b5b6bf964a520c8ea1fe3',
 '4ca50f407334236a60ef1258',
 '4cdb36c1958f236a15a7ab03',
 '53ed3b37498e4151087521a9',
 '49e2a407f964a52045621fe3',
 '4d4218cd607b6dcb31df08c6',
 '49dfb562f964a52001611fe3',
 '4eb1b6859adfb95b77765bf9',
 '4a9ac1b1f964a520813220e3',
 '4ad12c5ef964a5203ddd20e3',
 '4a453f9cf964a520f4a71fe3',
 '57f83f7acd10164c2ec1956f',
 '4c03f2fe39d476b0f5c530a7',
 '4c60c4a1de6920a111ed9664',
 '527f3d1711d2f7f001c656b2',
 '4a9578dff964a520562320e3',
 '58e7ed715f67173549fe6246',
 '49ee57f6f964a5204f681fe3',
 '4bddbf6be75c0f47f171c503',
 '5a888bc2c47cf91545daec08',
 '4a8da189f964a520501020e3',
 '58c470fd37da1d593431c33a',
 '4c41e1aee26920a1981e5fe7',
 '51070e83e4b0d92d935e04a3',
 '5a6b6047f427de038c51031c',
 '4ad89c0bf964a520d31221e3',
 '4a775512f964a5202ee41fe3',
 '4a3ad481f964a52057a01fe3',
 '4a9703def964a520fb2720e3',
 '5065c556e4b0a44a76b324ce',
 '49e37784f964a52083621fe3',
 '5131669e582f0b9471213351',
 '55d52901498ea18f871d5f9e',
 '4b191a58f964a520ffd723e3',
 '56f136ad498ecc5661aa49ce',
 '5ac2cec0419a9e0f679671f3',
 '49e27ed2f964a5201c621fe3',
 '59fd168ec21cb1401894e47e',
 '4b76e28df964a52021672ee3',
 '5b5b1e0bd4cc9800394e3256',
 '4283ee00f964a52091221fe3',
 '4acce0aaf964a520d8c920e3',
 '4a9c695cf964a520153720e3',
 '567b01d8498e9310602c8d10',
 '4b44d83ef964a520f4fd25e3',
 '522cb84211d20c34ef82194f',
 '49cf90f2f964a520ae5a1fe3',
 '49eabe84f964a52090661fe3',
 '4af5d847f964a520aefd21e3',
 '49e8d1a7f964a52097651fe3',
 '529fc55011d2e9632e382ae3']
In [12]:
#set up to pull the likes from the API based on venue ID

url_list = []
like_list = []
json_list = []

for i in venue_id_list:
    venue_url = 'https://api.foursquare.com/v2/venues/{}/likes?client_id={}&client_secret={}&v={}'.format(i, CLIENT_ID, CLIENT_SECRET, VERSION)
    url_list.append(venue_url)
for link in url_list:
    result = requests.get(link).json()
    likes = result['response']['likes']['count']
    like_list.append(likes)
print(like_list)
[151, 62, 60, 66, 28, 35, 98, 131, 268, 116, 35, 136, 81, 81, 101, 179, 62, 163, 162, 42, 68, 28, 32, 64, 50, 76, 63, 13, 4, 11, 71, 23, 16, 23, 31, 21, 71, 68, 30, 103, 108, 119, 10, 39, 11, 9, 44, 13, 48, 10, 45, 20, 31, 24, 20, 65, 72, 31, 18, 29, 29]
In [13]:
#double check that we did not lose any venues based on if likes were available

print(len(like_list))
print(len(venue_id_list))
61
61
Data Prep Intro
The thought process behind this is that likes are a proxy for quality. The more likes there are, the better the restaurant is. This might be incorrect but API call issues (how many I can use for free) holds me back from getting price / rating data. I will then bin this data into a quality categorical variables so we can cluster appropriately.

I am also going to create new categorical variables for the restaurants to better group them based on type of cuisine. This way you can look for good mexican food or now what type of food might be best to eat in Hoboken if you are new to the area.

Data Prep Part 1
Now let's start prepping our data for clustering. This will include combining data from different lists, creating new categorical data to be used, binning data and then encoding the data for clustering.

In [14]:
#let's make a copy of our initial dataframe just in case anything goes wrong

hoboken_venues = nearby_venues2.copy()
hoboken_venues.head()
Out[14]:
name	id	categories	lat	lng
1	Onieal's Restaurant & Bar	45e9482df964a52075431fe3	Pub	40.741608	-74.032304
4	Grand Vin	56d3b920498ec4e1c67c0907	Cocktail Bar	40.743209	-74.035099
6	O'Bagel	56daf06fcd107605ef3d86ea	Bagel Shop	40.743603	-74.029173
7	Ayame Hibachi & Sushi	4dbc9859f7b1ab37dd636d12	Sushi Restaurant	40.743105	-74.029213
8	Kung Fu Tea	57168865498e9517f09fa03d	Bubble Tea Shop	40.743375	-74.029450
Data Prep Part 2
Let's combine our list of likes into our dataframe

In [15]:
# add in the list of likes

hoboken_venues['total likes'] = like_list
hoboken_venues.head()
Out[15]:
name	id	categories	lat	lng	total likes
1	Onieal's Restaurant & Bar	45e9482df964a52075431fe3	Pub	40.741608	-74.032304	151
4	Grand Vin	56d3b920498ec4e1c67c0907	Cocktail Bar	40.743209	-74.035099	62
6	O'Bagel	56daf06fcd107605ef3d86ea	Bagel Shop	40.743603	-74.029173	60
7	Ayame Hibachi & Sushi	4dbc9859f7b1ab37dd636d12	Sushi Restaurant	40.743105	-74.029213	66
8	Kung Fu Tea	57168865498e9517f09fa03d	Bubble Tea Shop	40.743375	-74.029450	28
Data Prep 3
Let's look at our like data to set bins

In [16]:
# now let's bin total likes

print(hoboken_venues['total likes'].max())
print(hoboken_venues['total likes'].min())
print(hoboken_venues['total likes'].median())
print(hoboken_venues['total likes'].mean())
268
4
45.0
60.950819672131146
In [17]:
# let's visualize our total likes based on a histogram

import matplotlib.pyplot as plt
hoboken_venues['total likes'].hist(bins=4)
plt.show()

In [18]:
# what are the bins we want to use?

print(np.percentile(hoboken_venues['total likes'], 25))
print(np.percentile(hoboken_venues['total likes'], 50))
print(np.percentile(hoboken_venues['total likes'], 75))
24.0
45.0
76.0
In [19]:
# now we have our bin values so let's set them to the appropriate values
# less than 24, 24-45, 45-76, 76>
# poor, below avg, abv avg, great

poor = hoboken_venues['total likes']<=24
below_avg = hoboken_venues[(hoboken_venues['total likes']>24) & (hoboken_venues['total likes']<=45)]
abv_avg = hoboken_venues[(hoboken_venues['total likes']>45) & (hoboken_venues['total likes']<=76)]
great = hoboken_venues['total likes']>76
In [20]:
# let's set up a function that will re-categorize our restaurants based on likes

def conditions(s):
    if s['total likes']<=24:
        return 'poor'
    if s['total likes']<=45:
        return 'below avg'
    if s['total likes']<=76:
        return 'avg avg'
    if s['total likes']>76:
        return 'great'

hoboken_venues['total likes_cat']=hoboken_venues.apply(conditions, axis=1)
In [21]:
hoboken_venues
Out[21]:
name	id	categories	lat	lng	total likes	total likes_cat
1	Onieal's Restaurant & Bar	45e9482df964a52075431fe3	Pub	40.741608	-74.032304	151	great
4	Grand Vin	56d3b920498ec4e1c67c0907	Cocktail Bar	40.743209	-74.035099	62	avg avg
6	O'Bagel	56daf06fcd107605ef3d86ea	Bagel Shop	40.743603	-74.029173	60	avg avg
7	Ayame Hibachi & Sushi	4dbc9859f7b1ab37dd636d12	Sushi Restaurant	40.743105	-74.029213	66	avg avg
8	Kung Fu Tea	57168865498e9517f09fa03d	Bubble Tea Shop	40.743375	-74.029450	28	below avg
10	Karma Kafe	582dfc9565be5809f6a964ed	Indian Restaurant	40.742373	-74.029376	35	below avg
13	Court Street Bar & Restaurant	4a7eff1cf964a5206ff21fe3	American Restaurant	40.743322	-74.028615	98	great
15	Empire Coffee & Tea	49f37b88f964a520a26a1fe3	Coffee Shop	40.741375	-74.030515	131	great
16	Mamoun's Falafel	4d9368407b5ea1437d14c8b8	Falafel Restaurant	40.742303	-74.029465	268	great
17	Zack's Oak Bar & Restaurant	49f26862f964a520296a1fe3	American Restaurant	40.740640	-74.033826	116	great
18	Cafe Michelina	4a7b5b6bf964a520c8ea1fe3	Italian Restaurant	40.742278	-74.030218	35	below avg
19	Grimaldi's	4ca50f407334236a60ef1258	Pizza Place	40.741674	-74.029578	136	great
20	Robongi	4cdb36c1958f236a15a7ab03	Sushi Restaurant	40.742879	-74.029280	81	great
22	Bareburger	53ed3b37498e4151087521a9	Burger Joint	40.742694	-74.029070	81	great
23	Cucharamama	49e2a407f964a52045621fe3	South American Restaurant	40.740807	-74.034582	101	great
24	Cork City Pub	4d4218cd607b6dcb31df08c6	Pub	40.740105	-74.030868	179	great
25	Ben & Jerry's	49dfb562f964a52001611fe3	Ice Cream Shop	40.741430	-74.029484	62	avg avg
27	The Cuban	4eb1b6859adfb95b77765bf9	Cuban Restaurant	40.741012	-74.029732	163	great
29	Benny Tudino's	4a9ac1b1f964a520813220e3	Pizza Place	40.744198	-74.028896	162	great
33	Moran's Pub	4ad12c5ef964a5203ddd20e3	Bar	40.742553	-74.031192	42	below avg
34	Zafra	4a453f9cf964a520f4a71fe3	Cuban Restaurant	40.740659	-74.033802	68	avg avg
35	Tutta Pesca	57f83f7acd10164c2ec1956f	Seafood Restaurant	40.740163	-74.031284	28	below avg
36	Ultramarinos	4c03f2fe39d476b0f5c530a7	Latin American Restaurant	40.740620	-74.033490	32	below avg
37	Dozzino	4c60c4a1de6920a111ed9664	Pizza Place	40.744612	-74.035632	64	avg avg
38	Otto Strada	527f3d1711d2f7f001c656b2	Italian Restaurant	40.746604	-74.031161	50	avg avg
39	Illuzion	4a9578dff964a520562320e3	Japanese Restaurant	40.741229	-74.029582	76	avg avg
40	Bluestone Lane	58e7ed715f67173549fe6246	Coffee Shop	40.741601	-74.029410	63	avg avg
42	Hoboken Burrito	49ee57f6f964a5204f681fe3	Mexican Restaurant	40.741822	-74.031881	13	poor
43	Townhouse No 620	4bddbf6be75c0f47f171c503	Boutique	40.744128	-74.029070	4	poor
44	Jefferson’s Coffee	5a888bc2c47cf91545daec08	Coffee Shop	40.743368	-74.029135	11	poor
45	Vito's Italian Deli	4a8da189f964a520501020e3	Sandwich Place	40.746401	-74.028310	71	avg avg
46	Makai Poke Co	58c470fd37da1d593431c33a	Poke Place	40.742852	-74.029213	23	poor
47	Chicken Factory	4c41e1aee26920a1981e5fe7	Korean Restaurant	40.742975	-74.029083	16	poor
49	Midtown Philly Steaks	51070e83e4b0d92d935e04a3	Sandwich Place	40.742983	-74.029211	23	poor
50	Mamoun’s Falafel	5a6b6047f427de038c51031c	Falafel Restaurant	40.740090	-74.030374	31	below avg
51	Mr Wrap's	4ad89c0bf964a520d31221e3	American Restaurant	40.746308	-74.030003	21	poor
53	Margherita's	4a775512f964a5202ee41fe3	Italian Restaurant	40.746034	-74.028412	71	avg avg
54	Mikie Squared Bar & Grill	4a3ad481f964a52057a01fe3	Sports Bar	40.744032	-74.028956	68	avg avg
58	Maroon	4a9703def964a520fb2720e3	Tea Room	40.745307	-74.032359	30	below avg
59	Chipotle Mexican Grill	5065c556e4b0a44a76b324ce	Mexican Restaurant	40.739599	-74.029981	103	great
60	East LA	49e37784f964a52083621fe3	Mexican Restaurant	40.742509	-74.029496	108	great
61	The Stewed Cow	5131669e582f0b9471213351	Bar	40.742461	-74.036385	119	great
64	Flatbread Grill	55d52901498ea18f871d5f9e	Restaurant	40.742767	-74.029281	10	poor
65	Dunkin' Donuts	4b191a58f964a520ffd723e3	Donut Shop	40.744813	-74.028812	39	below avg
66	Pita Pit	56f136ad498ecc5661aa49ce	Sandwich Place	40.745729	-74.028582	11	poor
70	Sauced	5ac2cec0419a9e0f679671f3	Sandwich Place	40.739274	-74.030205	9	poor
74	Sri Thai	49e27ed2f964a5201c621fe3	Thai Restaurant	40.740103	-74.030955	44	below avg
75	Napoli’s Pizzeria	59fd168ec21cb1401894e47e	Pizza Place	40.739404	-74.034909	13	poor
76	Hoboken Hot Bagels	4b76e28df964a52021672ee3	Bagel Shop	40.744448	-74.028822	48	avg avg
78	Hidden Grounds Coffee	5b5b1e0bd4cc9800394e3256	Coffee Shop	40.745117	-74.030493	10	poor
79	Louise & Jerry's	4283ee00f964a52091221fe3	Dive Bar	40.740880	-74.029480	45	below avg
80	Bangkok City	4acce0aaf964a520d8c920e3	Thai Restaurant	40.741110	-74.029734	20	poor
81	Trattoria Saporito	4a9c695cf964a520153720e3	Italian Restaurant	40.740996	-74.029827	31	below avg
82	Frankie & Ava's Italian Eatery	567b01d8498e9310602c8d10	Italian Restaurant	40.739127	-74.030450	24	poor
83	Mario's Classic Pizza Café	4b44d83ef964a520f4fd25e3	Pizza Place	40.746364	-74.030152	20	poor
85	Finnegan's Pub	522cb84211d20c34ef82194f	Pub	40.746477	-74.032032	65	avg avg
87	Willie McBride's	49cf90f2f964a520ae5a1fe3	Bar	40.745133	-74.034497	72	avg avg
89	QDOBA Mexican Eats	49eabe84f964a52090661fe3	Mexican Restaurant	40.741413	-74.029926	31	below avg
93	H&S Giovanni's	4af5d847f964a520aefd21e3	Pizza Place	40.743508	-74.029017	18	poor
97	Farside	49e8d1a7f964a52097651fe3	Bar	40.743198	-74.029145	29	below avg
99	Pho Nomenon	529fc55011d2e9632e382ae3	Vietnamese Restaurant	40.742733	-74.029298	29	below avg
In [22]:
# let's star the process for re-categorizing the categories

hoboken_venues['categories'].unique()
Out[22]:
array(['Pub', 'Cocktail Bar', 'Bagel Shop', 'Sushi Restaurant',
       'Bubble Tea Shop', 'Indian Restaurant', 'American Restaurant',
       'Coffee Shop', 'Falafel Restaurant', 'Italian Restaurant',
       'Pizza Place', 'Burger Joint', 'South American Restaurant',
       'Ice Cream Shop', 'Cuban Restaurant', 'Bar', 'Seafood Restaurant',
       'Latin American Restaurant', 'Japanese Restaurant',
       'Mexican Restaurant', 'Boutique', 'Sandwich Place', 'Poke Place',
       'Korean Restaurant', 'Sports Bar', 'Tea Room', 'Restaurant',
       'Donut Shop', 'Thai Restaurant', 'Dive Bar', 'Vietnamese Restaurant'], dtype=object)
In [23]:
# let's create our new categories and create a function to apply those to our existing data


bars = ['Pub', 'Cocktail Bar', 'Bar', 'Dive Bar', 'Sports Bar']
other = ['Bagel Shop', 'Tea Room', 'Donut Shop', 'Coffee Shop', 'Bubble Tea Shop', 'Sandwich Place', 'Boutique', 'Ice Cream Shop']
euro_asia_indian_food = ['Falafel Restaurant', 'Korean Restaurant','Sushi Restaurant', 'Indian Restaurant', 'Japanese Restaurant', 'Poke Place', 'Thai Restaurant', 'Vietnamese Restaurant']
mex_southam_food = ['Cuban Restaurant', 'Mexican Restaurant', 'South American Restaurant', 'Latin American Restaurant']
american_food = ['Burger Joint', 'Restaurant', 'American Restaurant']
italian_food = ['Italian Restaurant', 'Seafood Restaurant', 'Pizza Place']

def conditions2(s):
    if s['categories'] in bars:
        return 'bars'
    if s['categories'] in other:
        return 'other'
    if s['categories'] in euro_asia_indian_food:
        return 'euro asia indian food'
    if s['categories'] in mex_southam_food:
        return 'mex southam food'
    if s['categories'] in american_food:
        return 'american food'
    if s['categories'] in italian_food:
        return 'italian food'

hoboken_venues['categories_new']=hoboken_venues.apply(conditions2, axis=1)
In [69]:
hoboken_venues
Out[69]:
name	id	categories	lat	lng	total likes	total likes_cat	Neighborhood	categories_new
1	Onieal's Restaurant & Bar	45e9482df964a52075431fe3	Pub	40.741608	-74.032304	151	great	Hoboken	bars
4	Grand Vin	56d3b920498ec4e1c67c0907	Cocktail Bar	40.743209	-74.035099	62	avg avg	Hoboken	bars
6	O'Bagel	56daf06fcd107605ef3d86ea	Bagel Shop	40.743603	-74.029173	60	avg avg	Hoboken	other
7	Ayame Hibachi & Sushi	4dbc9859f7b1ab37dd636d12	Sushi Restaurant	40.743105	-74.029213	66	avg avg	Hoboken	euro asia indian food
8	Kung Fu Tea	57168865498e9517f09fa03d	Bubble Tea Shop	40.743375	-74.029450	28	below avg	Hoboken	other
10	Karma Kafe	582dfc9565be5809f6a964ed	Indian Restaurant	40.742373	-74.029376	35	below avg	Hoboken	euro asia indian food
13	Court Street Bar & Restaurant	4a7eff1cf964a5206ff21fe3	American Restaurant	40.743322	-74.028615	98	great	Hoboken	american food
15	Empire Coffee & Tea	49f37b88f964a520a26a1fe3	Coffee Shop	40.741375	-74.030515	131	great	Hoboken	other
16	Mamoun's Falafel	4d9368407b5ea1437d14c8b8	Falafel Restaurant	40.742303	-74.029465	268	great	Hoboken	euro asia indian food
17	Zack's Oak Bar & Restaurant	49f26862f964a520296a1fe3	American Restaurant	40.740640	-74.033826	116	great	Hoboken	american food
18	Cafe Michelina	4a7b5b6bf964a520c8ea1fe3	Italian Restaurant	40.742278	-74.030218	35	below avg	Hoboken	italian food
19	Grimaldi's	4ca50f407334236a60ef1258	Pizza Place	40.741674	-74.029578	136	great	Hoboken	italian food
20	Robongi	4cdb36c1958f236a15a7ab03	Sushi Restaurant	40.742879	-74.029280	81	great	Hoboken	euro asia indian food
22	Bareburger	53ed3b37498e4151087521a9	Burger Joint	40.742694	-74.029070	81	great	Hoboken	american food
23	Cucharamama	49e2a407f964a52045621fe3	South American Restaurant	40.740807	-74.034582	101	great	Hoboken	mex southam food
24	Cork City Pub	4d4218cd607b6dcb31df08c6	Pub	40.740105	-74.030868	179	great	Hoboken	bars
25	Ben & Jerry's	49dfb562f964a52001611fe3	Ice Cream Shop	40.741430	-74.029484	62	avg avg	Hoboken	other
27	The Cuban	4eb1b6859adfb95b77765bf9	Cuban Restaurant	40.741012	-74.029732	163	great	Hoboken	mex southam food
29	Benny Tudino's	4a9ac1b1f964a520813220e3	Pizza Place	40.744198	-74.028896	162	great	Hoboken	italian food
33	Moran's Pub	4ad12c5ef964a5203ddd20e3	Bar	40.742553	-74.031192	42	below avg	Hoboken	bars
34	Zafra	4a453f9cf964a520f4a71fe3	Cuban Restaurant	40.740659	-74.033802	68	avg avg	Hoboken	mex southam food
35	Tutta Pesca	57f83f7acd10164c2ec1956f	Seafood Restaurant	40.740163	-74.031284	28	below avg	Hoboken	italian food
36	Ultramarinos	4c03f2fe39d476b0f5c530a7	Latin American Restaurant	40.740620	-74.033490	32	below avg	Hoboken	mex southam food
37	Dozzino	4c60c4a1de6920a111ed9664	Pizza Place	40.744612	-74.035632	64	avg avg	Hoboken	italian food
38	Otto Strada	527f3d1711d2f7f001c656b2	Italian Restaurant	40.746604	-74.031161	50	avg avg	Hoboken	italian food
39	Illuzion	4a9578dff964a520562320e3	Japanese Restaurant	40.741229	-74.029582	76	avg avg	Hoboken	euro asia indian food
40	Bluestone Lane	58e7ed715f67173549fe6246	Coffee Shop	40.741601	-74.029410	63	avg avg	Hoboken	other
42	Hoboken Burrito	49ee57f6f964a5204f681fe3	Mexican Restaurant	40.741822	-74.031881	13	poor	Hoboken	mex southam food
43	Townhouse No 620	4bddbf6be75c0f47f171c503	Boutique	40.744128	-74.029070	4	poor	Hoboken	other
44	Jefferson’s Coffee	5a888bc2c47cf91545daec08	Coffee Shop	40.743368	-74.029135	11	poor	Hoboken	other
45	Vito's Italian Deli	4a8da189f964a520501020e3	Sandwich Place	40.746401	-74.028310	71	avg avg	Hoboken	other
46	Makai Poke Co	58c470fd37da1d593431c33a	Poke Place	40.742852	-74.029213	23	poor	Hoboken	euro asia indian food
47	Chicken Factory	4c41e1aee26920a1981e5fe7	Korean Restaurant	40.742975	-74.029083	16	poor	Hoboken	euro asia indian food
49	Midtown Philly Steaks	51070e83e4b0d92d935e04a3	Sandwich Place	40.742983	-74.029211	23	poor	Hoboken	other
50	Mamoun’s Falafel	5a6b6047f427de038c51031c	Falafel Restaurant	40.740090	-74.030374	31	below avg	Hoboken	euro asia indian food
51	Mr Wrap's	4ad89c0bf964a520d31221e3	American Restaurant	40.746308	-74.030003	21	poor	Hoboken	american food
53	Margherita's	4a775512f964a5202ee41fe3	Italian Restaurant	40.746034	-74.028412	71	avg avg	Hoboken	italian food
54	Mikie Squared Bar & Grill	4a3ad481f964a52057a01fe3	Sports Bar	40.744032	-74.028956	68	avg avg	Hoboken	bars
58	Maroon	4a9703def964a520fb2720e3	Tea Room	40.745307	-74.032359	30	below avg	Hoboken	other
59	Chipotle Mexican Grill	5065c556e4b0a44a76b324ce	Mexican Restaurant	40.739599	-74.029981	103	great	Hoboken	mex southam food
60	East LA	49e37784f964a52083621fe3	Mexican Restaurant	40.742509	-74.029496	108	great	Hoboken	mex southam food
61	The Stewed Cow	5131669e582f0b9471213351	Bar	40.742461	-74.036385	119	great	Hoboken	bars
64	Flatbread Grill	55d52901498ea18f871d5f9e	Restaurant	40.742767	-74.029281	10	poor	Hoboken	american food
65	Dunkin' Donuts	4b191a58f964a520ffd723e3	Donut Shop	40.744813	-74.028812	39	below avg	Hoboken	other
66	Pita Pit	56f136ad498ecc5661aa49ce	Sandwich Place	40.745729	-74.028582	11	poor	Hoboken	other
70	Sauced	5ac2cec0419a9e0f679671f3	Sandwich Place	40.739274	-74.030205	9	poor	Hoboken	other
74	Sri Thai	49e27ed2f964a5201c621fe3	Thai Restaurant	40.740103	-74.030955	44	below avg	Hoboken	euro asia indian food
75	Napoli’s Pizzeria	59fd168ec21cb1401894e47e	Pizza Place	40.739404	-74.034909	13	poor	Hoboken	italian food
76	Hoboken Hot Bagels	4b76e28df964a52021672ee3	Bagel Shop	40.744448	-74.028822	48	avg avg	Hoboken	other
78	Hidden Grounds Coffee	5b5b1e0bd4cc9800394e3256	Coffee Shop	40.745117	-74.030493	10	poor	Hoboken	other
79	Louise & Jerry's	4283ee00f964a52091221fe3	Dive Bar	40.740880	-74.029480	45	below avg	Hoboken	bars
80	Bangkok City	4acce0aaf964a520d8c920e3	Thai Restaurant	40.741110	-74.029734	20	poor	Hoboken	euro asia indian food
81	Trattoria Saporito	4a9c695cf964a520153720e3	Italian Restaurant	40.740996	-74.029827	31	below avg	Hoboken	italian food
82	Frankie & Ava's Italian Eatery	567b01d8498e9310602c8d10	Italian Restaurant	40.739127	-74.030450	24	poor	Hoboken	italian food
83	Mario's Classic Pizza Café	4b44d83ef964a520f4fd25e3	Pizza Place	40.746364	-74.030152	20	poor	Hoboken	italian food
85	Finnegan's Pub	522cb84211d20c34ef82194f	Pub	40.746477	-74.032032	65	avg avg	Hoboken	bars
87	Willie McBride's	49cf90f2f964a520ae5a1fe3	Bar	40.745133	-74.034497	72	avg avg	Hoboken	bars
89	QDOBA Mexican Eats	49eabe84f964a52090661fe3	Mexican Restaurant	40.741413	-74.029926	31	below avg	Hoboken	mex southam food
93	H&S Giovanni's	4af5d847f964a520aefd21e3	Pizza Place	40.743508	-74.029017	18	poor	Hoboken	italian food
97	Farside	49e8d1a7f964a52097651fe3	Bar	40.743198	-74.029145	29	below avg	Hoboken	bars
99	Pho Nomenon	529fc55011d2e9632e382ae3	Vietnamese Restaurant	40.742733	-74.029298	29	below avg	Hoboken	euro asia indian food
Data Prep Part 4
Now let's create dummy variables for our total likes and categories so we can cluster

In [24]:
# one hot encoding
hoboken_onehot = pd.get_dummies(hoboken_venues[['categories_new', 'total likes_cat']], prefix="", prefix_sep="")

# add neighborhood column back to dataframe
hoboken_onehot['Name'] = hoboken_venues['name'] 

# move neighborhood column to the first column
fixed_columns = [hoboken_onehot.columns[-1]] + list(hoboken_onehot.columns[:-1])
hoboken_onehot = hoboken_onehot[fixed_columns]

hoboken_onehot.head()
Out[24]:
Name	american food	bars	euro asia indian food	italian food	mex southam food	other	avg avg	below avg	great	poor
1	Onieal's Restaurant & Bar	0	1	0	0	0	0	0	0	1	0
4	Grand Vin	0	1	0	0	0	0	1	0	0	0
6	O'Bagel	0	0	0	0	0	1	1	0	0	0
7	Ayame Hibachi & Sushi	0	0	1	0	0	0	1	0	0	0
8	Kung Fu Tea	0	0	0	0	0	1	0	1	0	0
Clustering Part 1
Now let's run our k-means clustering algo to get our labels

In [25]:
cluster_df = hoboken_onehot.drop('Name', axis=1)

k_clusters = 4

# run k-means clustering
kmeans = KMeans(n_clusters=k_clusters, random_state=0).fit(cluster_df)

# check cluster labels generated for each row in the dataframe
kmeans.labels_[0:10]
Out[25]:
array([2, 3, 3, 3, 1, 1, 2, 2, 2, 2], dtype=int32)
Clustering Part 2
Let's add our cluster labels back into our original dataframe.

In [26]:
hoboken_venues['label'] = kmeans.labels_
hoboken_venues.head()
Out[26]:
name	id	categories	lat	lng	total likes	total likes_cat	categories_new	label
1	Onieal's Restaurant & Bar	45e9482df964a52075431fe3	Pub	40.741608	-74.032304	151	great	bars	2
4	Grand Vin	56d3b920498ec4e1c67c0907	Cocktail Bar	40.743209	-74.035099	62	avg avg	bars	3
6	O'Bagel	56daf06fcd107605ef3d86ea	Bagel Shop	40.743603	-74.029173	60	avg avg	other	3
7	Ayame Hibachi & Sushi	4dbc9859f7b1ab37dd636d12	Sushi Restaurant	40.743105	-74.029213	66	avg avg	euro asia indian food	3
8	Kung Fu Tea	57168865498e9517f09fa03d	Bubble Tea Shop	40.743375	-74.029450	28	below avg	other	1
Clustering Part 3
Now let's visualize what our clusters look like for Hoboken.

In [27]:
map_clusters = folium.Map(location=[latitude, longitude], zoom_start=11)

# set color scheme for the clusters
x = np.arange(k_clusters)
ys = [i+x+(i*x)**2 for i in range(k_clusters)]
colors_array = cm.rainbow(np.linspace(0, 1, len(ys)))
rainbow = [colors.rgb2hex(i) for i in colors_array]

# add markers to the map
markers_colors = []
for lat, lon, poi, cluster in zip(hoboken_venues['lat'], hoboken_venues['lng'], hoboken_venues['name'], hoboken_venues['label']):
    label = folium.Popup(str(poi) + ' Cluster ' + str(cluster), parse_html=True)
    folium.CircleMarker(
        [lat, lon],
        radius=5,
        popup=label,
        color=rainbow[cluster-1],
        fill=True,
        fill_color=rainbow[cluster-1],
        fill_opacity=0.7).add_to(map_clusters)
       
map_clusters
Out[27]:
Clustering Part 4
Now let's see what is in each of our clusters

Cluster 1
characteristics
Poor quality food
Mostly Italian food or other
In [28]:
hoboken_venues.loc[hoboken_venues['label']==0]
Out[28]:
name	id	categories	lat	lng	total likes	total likes_cat	categories_new	label
42	Hoboken Burrito	49ee57f6f964a5204f681fe3	Mexican Restaurant	40.741822	-74.031881	13	poor	mex southam food	0
43	Townhouse No 620	4bddbf6be75c0f47f171c503	Boutique	40.744128	-74.029070	4	poor	other	0
44	Jefferson’s Coffee	5a888bc2c47cf91545daec08	Coffee Shop	40.743368	-74.029135	11	poor	other	0
46	Makai Poke Co	58c470fd37da1d593431c33a	Poke Place	40.742852	-74.029213	23	poor	euro asia indian food	0
47	Chicken Factory	4c41e1aee26920a1981e5fe7	Korean Restaurant	40.742975	-74.029083	16	poor	euro asia indian food	0
49	Midtown Philly Steaks	51070e83e4b0d92d935e04a3	Sandwich Place	40.742983	-74.029211	23	poor	other	0
51	Mr Wrap's	4ad89c0bf964a520d31221e3	American Restaurant	40.746308	-74.030003	21	poor	american food	0
64	Flatbread Grill	55d52901498ea18f871d5f9e	Restaurant	40.742767	-74.029281	10	poor	american food	0
66	Pita Pit	56f136ad498ecc5661aa49ce	Sandwich Place	40.745729	-74.028582	11	poor	other	0
70	Sauced	5ac2cec0419a9e0f679671f3	Sandwich Place	40.739274	-74.030205	9	poor	other	0
75	Napoli’s Pizzeria	59fd168ec21cb1401894e47e	Pizza Place	40.739404	-74.034909	13	poor	italian food	0
78	Hidden Grounds Coffee	5b5b1e0bd4cc9800394e3256	Coffee Shop	40.745117	-74.030493	10	poor	other	0
80	Bangkok City	4acce0aaf964a520d8c920e3	Thai Restaurant	40.741110	-74.029734	20	poor	euro asia indian food	0
82	Frankie & Ava's Italian Eatery	567b01d8498e9310602c8d10	Italian Restaurant	40.739127	-74.030450	24	poor	italian food	0
83	Mario's Classic Pizza Café	4b44d83ef964a520f4fd25e3	Pizza Place	40.746364	-74.030152	20	poor	italian food	0
93	H&S Giovanni's	4af5d847f964a520aefd21e3	Pizza Place	40.743508	-74.029017	18	poor	italian food	0
Cluster 2
characteristics
below average quality food
Mostly Europe / Asia inspired food
In [29]:
hoboken_venues.loc[hoboken_venues['label']==1]
Out[29]:
name	id	categories	lat	lng	total likes	total likes_cat	categories_new	label
8	Kung Fu Tea	57168865498e9517f09fa03d	Bubble Tea Shop	40.743375	-74.029450	28	below avg	other	1
10	Karma Kafe	582dfc9565be5809f6a964ed	Indian Restaurant	40.742373	-74.029376	35	below avg	euro asia indian food	1
18	Cafe Michelina	4a7b5b6bf964a520c8ea1fe3	Italian Restaurant	40.742278	-74.030218	35	below avg	italian food	1
33	Moran's Pub	4ad12c5ef964a5203ddd20e3	Bar	40.742553	-74.031192	42	below avg	bars	1
35	Tutta Pesca	57f83f7acd10164c2ec1956f	Seafood Restaurant	40.740163	-74.031284	28	below avg	italian food	1
36	Ultramarinos	4c03f2fe39d476b0f5c530a7	Latin American Restaurant	40.740620	-74.033490	32	below avg	mex southam food	1
50	Mamoun’s Falafel	5a6b6047f427de038c51031c	Falafel Restaurant	40.740090	-74.030374	31	below avg	euro asia indian food	1
58	Maroon	4a9703def964a520fb2720e3	Tea Room	40.745307	-74.032359	30	below avg	other	1
65	Dunkin' Donuts	4b191a58f964a520ffd723e3	Donut Shop	40.744813	-74.028812	39	below avg	other	1
74	Sri Thai	49e27ed2f964a5201c621fe3	Thai Restaurant	40.740103	-74.030955	44	below avg	euro asia indian food	1
79	Louise & Jerry's	4283ee00f964a52091221fe3	Dive Bar	40.740880	-74.029480	45	below avg	bars	1
81	Trattoria Saporito	4a9c695cf964a520153720e3	Italian Restaurant	40.740996	-74.029827	31	below avg	italian food	1
89	QDOBA Mexican Eats	49eabe84f964a52090661fe3	Mexican Restaurant	40.741413	-74.029926	31	below avg	mex southam food	1
97	Farside	49e8d1a7f964a52097651fe3	Bar	40.743198	-74.029145	29	below avg	bars	1
99	Pho Nomenon	529fc55011d2e9632e382ae3	Vietnamese Restaurant	40.742733	-74.029298	29	below avg	euro asia indian food	1
Cluster 3
characteristics
High quality food
Mostly Mexican and South American food
In [30]:
hoboken_venues.loc[hoboken_venues['label']==2]
Out[30]:
name	id	categories	lat	lng	total likes	total likes_cat	categories_new	label
1	Onieal's Restaurant & Bar	45e9482df964a52075431fe3	Pub	40.741608	-74.032304	151	great	bars	2
13	Court Street Bar & Restaurant	4a7eff1cf964a5206ff21fe3	American Restaurant	40.743322	-74.028615	98	great	american food	2
15	Empire Coffee & Tea	49f37b88f964a520a26a1fe3	Coffee Shop	40.741375	-74.030515	131	great	other	2
16	Mamoun's Falafel	4d9368407b5ea1437d14c8b8	Falafel Restaurant	40.742303	-74.029465	268	great	euro asia indian food	2
17	Zack's Oak Bar & Restaurant	49f26862f964a520296a1fe3	American Restaurant	40.740640	-74.033826	116	great	american food	2
19	Grimaldi's	4ca50f407334236a60ef1258	Pizza Place	40.741674	-74.029578	136	great	italian food	2
20	Robongi	4cdb36c1958f236a15a7ab03	Sushi Restaurant	40.742879	-74.029280	81	great	euro asia indian food	2
22	Bareburger	53ed3b37498e4151087521a9	Burger Joint	40.742694	-74.029070	81	great	american food	2
23	Cucharamama	49e2a407f964a52045621fe3	South American Restaurant	40.740807	-74.034582	101	great	mex southam food	2
24	Cork City Pub	4d4218cd607b6dcb31df08c6	Pub	40.740105	-74.030868	179	great	bars	2
27	The Cuban	4eb1b6859adfb95b77765bf9	Cuban Restaurant	40.741012	-74.029732	163	great	mex southam food	2
29	Benny Tudino's	4a9ac1b1f964a520813220e3	Pizza Place	40.744198	-74.028896	162	great	italian food	2
59	Chipotle Mexican Grill	5065c556e4b0a44a76b324ce	Mexican Restaurant	40.739599	-74.029981	103	great	mex southam food	2
60	East LA	49e37784f964a52083621fe3	Mexican Restaurant	40.742509	-74.029496	108	great	mex southam food	2
61	The Stewed Cow	5131669e582f0b9471213351	Bar	40.742461	-74.036385	119	great	bars	2
Cluster 4
characteristics
Above average quality food
Mostly Bars
In [31]:
hoboken_venues.loc[hoboken_venues['label']==3]
Out[31]:
name	id	categories	lat	lng	total likes	total likes_cat	categories_new	label
4	Grand Vin	56d3b920498ec4e1c67c0907	Cocktail Bar	40.743209	-74.035099	62	avg avg	bars	3
6	O'Bagel	56daf06fcd107605ef3d86ea	Bagel Shop	40.743603	-74.029173	60	avg avg	other	3
7	Ayame Hibachi & Sushi	4dbc9859f7b1ab37dd636d12	Sushi Restaurant	40.743105	-74.029213	66	avg avg	euro asia indian food	3
25	Ben & Jerry's	49dfb562f964a52001611fe3	Ice Cream Shop	40.741430	-74.029484	62	avg avg	other	3
34	Zafra	4a453f9cf964a520f4a71fe3	Cuban Restaurant	40.740659	-74.033802	68	avg avg	mex southam food	3
37	Dozzino	4c60c4a1de6920a111ed9664	Pizza Place	40.744612	-74.035632	64	avg avg	italian food	3
38	Otto Strada	527f3d1711d2f7f001c656b2	Italian Restaurant	40.746604	-74.031161	50	avg avg	italian food	3
39	Illuzion	4a9578dff964a520562320e3	Japanese Restaurant	40.741229	-74.029582	76	avg avg	euro asia indian food	3
40	Bluestone Lane	58e7ed715f67173549fe6246	Coffee Shop	40.741601	-74.029410	63	avg avg	other	3
45	Vito's Italian Deli	4a8da189f964a520501020e3	Sandwich Place	40.746401	-74.028310	71	avg avg	other	3
53	Margherita's	4a775512f964a5202ee41fe3	Italian Restaurant	40.746034	-74.028412	71	avg avg	italian food	3
54	Mikie Squared Bar & Grill	4a3ad481f964a52057a01fe3	Sports Bar	40.744032	-74.028956	68	avg avg	bars	3
76	Hoboken Hot Bagels	4b76e28df964a52021672ee3	Bagel Shop	40.744448	-74.028822	48	avg avg	other	3
85	Finnegan's Pub	522cb84211d20c34ef82194f	Pub	40.746477	-74.032032	65	avg avg	bars	3
87	Willie McBride's	49cf90f2f964a520ae5a1fe3	Bar	40.745133	-74.034497	72	avg avg	bars	3
In [ ]:
