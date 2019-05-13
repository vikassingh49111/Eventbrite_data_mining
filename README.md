# Eventbrite_data_mining
Pull data from eventbrite website
# -*- coding: utf-8 -*-
from __future__ import unicode_literals

import csv
import sys
reload(sys)
import requests
import time
import codecs
import datetime
from bs4 import BeautifulSoup
import pandas as pd
import os
import warnings
warnings.filterwarnings("ignore", category=UserWarning, module='bs4')

UTF8Writer = codecs.getwriter('utf8')
default_charset = 'UTF-8'
sys.setdefaultencoding("utf-8")
sys.stdout = UTF8Writer(sys.stdout)


class fetching_eventbritedata():
    cont = 0
    con = 0
    try:
        while cont <= con:

            url = 'https://www.eventbriteapi.com/v3/events/search/?location.address=toronto&expand=category,venue,organizer&token=X5DQJG5DKCWS6QWPRUH4&page=' + str(cont)

            abc = requests.get(url)
            abcd = abc.json()

            for item in abcd['events']:

                df = pd.DataFrame()

                df['subcategory_id'] = [item['subcategory_id']]

                if item['category_id'] is not None:

                        df['category_id'] = [
                            item['category_id'].replace('101', '2').replace('102', '34').replace('103', '21').replace('104','20')
                               .replace('105', '1').replace('106', '8').replace('107', '14').replace('108', '32').replace('109', '23')
                                .replace('110', '10').replace('111', '4').replace('112', '13').replace('113', '16').replace('114', '28')
                                .replace('115', '25').replace('116', '31').replace('117', '17').replace('118', '3').replace('119', '15')
                                .replace('199', '33')]

                else:
                    df['category_id'] = None


                if item['venue']['name'] is not None:
                    df['venue_name'] = [BeautifulSoup(item['venue']['name'] .replace("=", " ").lstrip()).get_text().encode('ascii', errors='ignore')]
                else:
                    df['venue_name'] = None

                df['address'] = [item['venue']['address']['localized_address_display']]
                df['lat'] = [item['venue']['latitude']]
                df['lon'] = [item['venue']['longitude']]
                df['start_date'] = [item['start']['utc'].replace("T", "  ").replace("Z", "")]

                df['city'] = [item['venue']['address']['city']]
                df['state'] = [item['venue']['address']['region']]

                df['url'] = [item['url']]

                if item['organizer']['name'] is not None:
                    df['group_name'] = [BeautifulSoup(item['organizer']['name'] .replace("=", " ").lstrip()).get_text().encode('ascii', errors='ignore')]
                else:
                    df['group_name'] = None

                if item['name']['text'] is not None:
                    df['event_name'] = [BeautifulSoup(item['name']['text'] .replace("=", " ").lstrip()).get_text().encode('ascii', errors='ignore')]
                else:
                    df['event_name'] = None

                if item['description']['text'] is not None:
                    df['disc'] = [BeautifulSoup(item['description']['text'] .replace("=", " ").lstrip()).get_text().encode('ascii', errors='ignore')]
                else:
                    df['disc'] = None

                df['end_date'] = [item['end']['utc'].replace("T", "  ").replace("Z", "")]

                try:
                    if item['logo']['original']['url'] is not None:
                        df['image'] = [item['logo']['original']['url']]
                    else:
                        df['image'] = None
                except Exception, e:
                    df['image'] = None

                conti = abcd['pagination']['page_number']
                con = abcd['pagination']['page_count']
                if conti== con:
                    break
                cont = conti+1
                with open("F:/raweventdata.csv", 'a') as f:
                    df.to_csv(f, header=False, index=False)

    except Exception, e:
        pass

def cleaning_data():
    events = ['subcategory', 'category', 'venue_name', 'address', 'lat', 'lon', 'start_date', 'city', 'state', 'url','group_name', 'event_name', 'disc', 'end_date', 'image']
    events_data = pd.read_csv('F:/raweventdata.csv', names=events)

    events_data.loc[events_data['subcategory'] == 15001, 'category'] = 6
    events_data.loc[events_data['subcategory'] == 17001, 'category'] = 30
    events_data.loc[events_data['subcategory'] == 17002, 'category'] = 26
    events_data.loc[events_data['subcategory'] == 19001, 'category'] = 29
    events_data.loc[events_data['subcategory'] == 19002, 'category'] = 11
    events_data.loc[events_data['subcategory'] == 19004, 'category'] = 27
    events_data.loc[events_data['subcategory'] == 19006, 'category'] = 18
    events_data.loc[events_data['subcategory'] == 5004, 'category'] = 5
    events_data.loc[events_data['subcategory'] == 5003, 'category'] = 5
    events_data.loc[events_data['subcategory'] == 5009, 'category'] = 36
    events_data.loc[events_data['subcategory'] == 13004, 'category'] = 12
    events_data.loc[events_data['subcategory'] == 4004, 'category'] = 11
    events_data.loc[events_data['subcategory'] == 14009, 'category'] = 22
    events_data.loc[events_data['subcategory'] == 13003, 'category'] = 4
    events_data.loc[events_data['subcategory'] == 13002, 'category'] = 4
    events_data.loc[events_data['subcategory'] == 13001, 'category'] = 4
    events_data.loc[events_data['subcategory'] == 8001, 'category'] = 9
    events_data.loc[events_data['subcategory'] == 8002, 'category'] = 9
    events_data.loc[events_data['subcategory'] == 8003, 'category'] = 9
    events_data.loc[events_data['subcategory'] == 8019, 'category'] = 9
    events_data.loc[events_data['subcategory'] == 8020, 'category'] = 9
    events_data.loc[events_data['category'] == 120, 'category'] = 33

    events_data['category']  = events_data['category'].fillna(33)

    with open("F:/Eventdata.csv", 'a') as f:
        events_data.to_csv(f, header = False,index=False, columns=['category', 'venue_name', 'address', 'lat', 'lon', 'start_date', 'city', 'state',
                                                                       'url','group_name', 'event_name', 'disc', 'end_date', 'image'])


def meetupdata():
    cities = [("Toronto", "ON"),("North York", "ON"),("Mississauga", "ON"),("Scarborough", "ON"),("Etobicoke", "ON"),("Brampton", "ON"),("Burlington", "ON"),("Oakville", "ON"),
              ("Hamilton", "ON"),("Oshawa", "ON"),("Richmond Hill", "ON"),("Kitchener", "ON"),("Markham", "ON"),("York", "ON"),("East York", "ON"),("Thornhill", "ON"),
              ("Guelph", "ON"),("Whitby", "ON"),("Barrie", "ON"),("Pickering", "ON"),("Ajax", "ON"),("Newmarket", "ON"),("Aurora", "ON"),("Woodbridge", "ON"),
              ("Orangeville", "ON"),("Cambridge", "ON"),("Brantford", "ON"),("Tonawanda", "ON"),("Milton", "ON"),("Bowmanville", "ON"),("Georgetown", "ON"),
              ("Concord", "ON"),("Lockport", "ON"),("Maple", "ON"),("Dundas", "ON")]

    for category in range(36):
        category = category + 1
        try:
            for (city, state) in cities:
                per_page = 200
                results_we_got = per_page
                offset = 0
                api_key = "a177256417426757e459746329598"

                while (results_we_got == per_page):

                    response = get_results(
                        {"sign": "true", "country": "ca", "city": city, "state": state, "category":category,"radius": 10, "key": api_key,
                         "page": per_page, "offset": offset})
                    time.sleep(1)
                    offset += 1

                    results_we_got = response['meta']['count']
                    for venue in response['results']:
                        group = ""

                        try:

                            if "group"  in venue:
                                address_1 = venue['venue']['address_1']
                            else:
                                address_1 = ['NA']

                        except Exception:
                            pass

                        try:

                            if "group"  in venue:
                                lat = venue['venue']['lat']
                            else:
                                lat = ['NA']

                        except Exception:
                            pass

                        try:

                            if "group"  in venue:
                                lon = venue['venue']['lon']
                            else:
                                lon = ['NA']

                        except Exception:
                            pass

                        try:
                            if "group"  in venue:
                                name = venue['venue']['name']
                        except Exception:
                            pass


                        if "group"  in venue:
                                group = venue['group']['name']

                        date = "error"

                        try:
                            date = datetime.datetime.utcfromtimestamp(venue['time']/1000).strftime('%Y-%m-%d %H:%M:%S')
                        except Exception, e:
                            print 'time not available, ', venue['time']


                        df = pd.DataFrame()

                        df['category'] = [category]

                        df['venue_name'] = [name]
                        df['address'] = [address_1]
                        df['lat'] = [lat]
                        df['lon'] = [lon]
                        df['date'] = [date]
                        df['city'] = [city]
                        df['state'] = [state]
                        df['url'] = [venue['event_url']]
                        df['group_name'] =[BeautifulSoup(group.replace("="," ").lstrip()).get_text().encode('ascii', errors='ignore')]
                        df['event_name']= [BeautifulSoup(venue['name'].replace("="," ").lstrip()).get_text().encode('ascii', errors='ignore')]
                        df['disc'] = [BeautifulSoup(venue['description'].replace("=", "").lstrip()).get_text().encode('ascii', errors='ignore')]

                        with open("F:/Eventdata.csv", 'a') as f:
                            df.to_csv(f,header=False, index= False)

        except Exception:
            pass

def get_results(params):
    request = requests.get("http://api.meetup.com//2/open_events", params=params)
    data = request.json()
    return data


if __name__ == "__main__":
    meetupdata()
    fetching_eventbritedata()
    cleaning_data()
    os.remove("F:/raweventdata.csv")
