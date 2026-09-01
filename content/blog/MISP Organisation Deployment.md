+++
title = "MISP Organisation Deployment"
date = "2026-09-01T16:00:13+01:00"
#dateFormat = "2006-01-02" # This value can be configured for per-post date formatting
author = ""
cover = ""
tags = ["project", "Short read"]
keywords = ["MISP","Threat Intel", "Deployment","Hosted"]
description = "Small write-up around work done during my day job in which I deployed MISP internally on our corporate environment. This is a small write-up involving the actions taken by myself as well as other in the organisation."
showFullContent = false
readingTime = true
hideComments = false
+++

## MISP - Malware information sharing platform - 01/09/26

It's been a while since I last updated this, with Summer holidays and busy social life I've not really had much time to do some write-ups of the work that I've completed.  First item on the chopping block for this is a piece of work that I completed in June of this year alongside some of my colleagues at work.  This involved a local deployment of MISP (Malware Information Sharing Platform) a threat intel platform used to collate attributes and correlate items seen in incidents and near misses. As this was a bit of work that was done for my job there will be elements that I cannot show or speak about but the general principles are the same.

As per MISPs installation guide documentation the decision was made to make use of an Ubuntu 24.04 server to host the MISP instance on. As this was done within the managed environment, a ticket was raised to our internal infrastructure teams to request this virtual server to be spun up. Once this was created alongside the usual hardening processes that are completed for new server builds, another task was completed to pull the installer from MISPs github repo:

``` Bash
#Installation command used
wget --no-cache -O /tmp/INSTALL.sh https://raw.githubusercontent.com/MISP/MISP/2.4/INSTALL/INSTALL.sh
bash /tmp/INSTALL.sh
``` 

Following the initial install that was completed, I got access to the server to take a look at the configuration of the platform.  This meant changing default admin credentials, adding in new organisations and correctly permissioning the required users to get access to the system.  Once the platform looked and behaved as we were looking it to internally we looked to the next step of the process which was attaching an SSL/TLS certificate to secure the traffic to and from the server.  To do this we created an internal certificate as there was no requirement for this to be accessible outside our corporate network. To save typing out the full process of what this entailed I've included the link to the [WEBSITE](https://www.ssdnodes.com/blog/create-a-self-signed-certificate-on-ubuntu-for-apache-and-nginx/) that I followed along with to create it.  As certificate handling is a task that is normally handled by our internal infrastructure team this was an interesting little exercise to try out myself for once and was a lot easier than expected.

With the certificate now in place, I wanted to make sure that the DNS A record was included locally so that we could access the MISP instance without having to remember the IP address for the server everytime we wanted to add or review any of the items included in the platform. As the DNS records for the organisation are managed by our internal Networks team this made it even easier for this process as all that was required to be done was raising a ticket with the DNS Hostname and the A Record(IP) to the team and get comfortable until it had been processed.

With the platform deployed and reachable in a convenient matter for my team, I started to look at importing feeds that provided information relevant to healthcare and public bodies. These were imported from MISPs default feeds found in their [GITHUB REPO](https://github.com/MISP/MISP/blob/2.4/app/files/feed-metadata/defaults.json) and sorted through to apply enough to provide information without killing the server by trying to import too much information at once.  From the list imported there were 3 that were enabled to pull information into the platform;

- URLhaus Malware URLs
- Malware Bazaar
- Threatfox

As I work within the NHS there are resources that I can use to get information relevant to the healthcare environment that would be worth importing into our MISP instance. NHS England has a feed of cyber alerts that it publishs using and xml file that we were lookign to export and convert to JSON to be imported into the MISP platform.

``` Python
import requests
import feedparser
import json
import time
import uuid
from datetime import datetime, timezone
from urllib.parse import urlparse

def fetch_rss(url):
    """Fetch RSS feed content from a URL."""
    try:
        response = requests.get(url, timeout=10)
        response.raise_for_status()
        return response.text
    except requests.RequestException as e:
        raise SystemExit(f"Error fetching RSS feed: {e}")

def rss_to_misp_json(rss_content, org_name="RSS Import", org_uuid=str(uuid.uuid7()),threat_level=2):
    """
    Convert RSS XML content to MISP JSON format.
    threat_level: 1=High, 2=Medium, 3=Low, 4=Undefined
    """
    feed = feedparser.parse(rss_content)
    if not feed.entries:
        raise ValueError("No entries found in RSS feed.")

    misp_events = []
    for entry in feed.entries:

        raw_date = entry.published

        dt = datetime.strptime(raw_date, "%a, %d %b %Y %H:%M:%S %Z")
        dt = dt.replace(tzinfo=timezone.utc)

        pub_date = int(dt.timestamp())

        event = {
            "Event": {
                "analysis": "0",
                "date": datetime.now().strftime("%Y-%m-%d"),
                "extends_uuid": "",
                "info": entry.get("title", "No title"),
                "publish_timestamp": pub_date,
                "published": True,
                "threat_level_id": threat_level,
                "timestamp": int(datetime.now(timezone.utc).timestamp()),
                "uuid": str(uuid.uuid7()), 
                "Orgc": {
                    "name": org_name,
                    "uuid": org_uuid
                },
                "Tag": [
                {
                    "colour": "#F56127",
                    "local": False,
                    "name": "RSS Feed Import"
                }],
                "Attribute":[
                ]
            }
        }
        if "description" in entry:
            description = entry.description
        else:
            description = "No description provided"

        # Add link as an attribute
        if "link" in entry:
            event["Event"]["Attribute"].append({
                "category": "External analysis",
                "comment": description,
                "deleted": False,
                "disable_correlation": False,
                "timestamp": pub_date,
                "to_ids": True,
                "type": "url",
                "value": entry.link,
                "uuid": str(uuid.uuid7())
            })

        misp_events.append(event)

    return misp_events

def save_json(data, filename):
    """Save JSON data to a file."""
    with open(filename, "w", encoding="utf-8") as f:
        json.dump(data, f, indent=2, ensure_ascii=False)

if __name__ == "__main__":
    rss_url = "https://digital.nhs.uk/feed/cyber-alerts-feed.xml"  # Replace with your RSS feed URL
    rss_content = fetch_rss(rss_url)
    misp_data = rss_to_misp_json(rss_content)
    save_json(misp_data, "misp_feed.json")
    print("MISP JSON feed saved as misp_feed.json")
``` 
The above python script allowed for that JSON file to be created from the XML feed in a format that could be used to upload into MISP. Within that script it also tagged the items to show that they came from the RSS feed import allowing them to be distinguished from other sources within the platform.  Once these sources were in the platform there was a bit of manual work to double check the links to the reports attached to these alerts to make sure that any IoCs were included to help draw correlations in any internal alerts raised on the platform.

This is a bit of a whirlwind tour of the work that was done around this and makes it seem like this was completed in 20 minutes when really it took a good few days to get working due to some problems that I can't really remember at this point. 
BUT with this platform now fully deployed in our environment we are populating it with more and more data to get a better threat intel specific to what we see internally as well as larger campaigns.  As this exercise matures we will hopefully see more and more value generated from this exercise.

Maybe one of these days I'll deploy an instance on my home network to add example of what the platform looks like and the work that I'd done around this but for the time being it will just have to be the loose verbal desriptions that will have to do.

Over the past little while I've been doing some work around renovating my home network to better set up testing environments and have been using a very introductory friendly suite with UniFi OS now that I've bought my Dream Router 7.  I think it will be the subject of my next post and I can get a lot more into the weeds with that one so if you care at all then keep an eye out.

Cheers,
Jack
:)