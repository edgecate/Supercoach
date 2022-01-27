# Supercoach
> Scrape data from the Supercoach website using Python for easier player analysis.

Watch the full YT tutorial:

[![IMAGE ALT TEXT](http://img.youtube.com/vi/WZXVkqavd0w/0.jpg)](http://www.youtube.com/watch?v=WZXVkqavd0w "How to scrape the Supercoach website")

```python
import requests
from bs4 import BeautifulSoup
import json
from oauthlib.oauth2 import LegacyApplicationClient
from requests_oauthlib import OAuth2Session

def get_sc_token():
    # Credentials to generate token
    client_id = 'INSERT_YOUR_CLIENT_ID'
    client_secret = ''
    get_token_url = 'https://supercoach.heraldsun.com.au/2019/api/afl/classic/v1/access_token'

    # Supercoach credentials
    sc_user = 'INSERT_YOUR_SC_USERNAME'
    sc_pass = 'INSERT_YOUR_SC_PASSWORD'

    # Create token
    oauth = OAuth2Session(client=LegacyApplicationClient(client_id=client_id))
    token = oauth.fetch_token(token_url=get_token_url,
            username=sc_user, password=sc_pass, client_id=client_id,
            client_secret=client_secret)

    # Get token value from the Access_Token key
    sc_token = token["access_token"]
    return sc_token

def download_stats():
    # Get a token
    sc_token = get_sc_token()

    # Append token to the end of the Stats Centre URL
    sc_url = 'https://supercoach.heraldsun.com.au/afl/draft/statscentre?access_token=' + sc_token

    # Create text file to store AFL stats
    file_output = open("stats.txt", "w")

    # HTTP request to Stats Centre URL
    res = requests.get(sc_url)

    # Parse the response as HTML using the BeautifulSoup library
    soup = BeautifulSoup(res.text, 'html.parser')

    # Find the start and end position of the data which is stored in the researchGridData variable
    start_id = "var researchGridData = "
    end_id = "}]"
    stat_start = str(soup).find(start_id) + len(start_id)
    soup_len = len(str(soup))
    stat_end = str(soup)[stat_start:soup_len].find(end_id) + len(end_id) + stat_start

    # Format it to JSON
    raw_stats = str("{\n\"researchGridData\": " + str(soup)[stat_start:stat_end] + "}").encode('utf8')

    # Parse to JSON to make extracting key values much easier
    json_stats = json.loads(raw_stats)

    # Write to file
    # Starting with the heading
    file_output.write("First Name|Last Name|Pos1|Pos2|Team|Total Pts|Rds|Rd Pts|Avg|Avg 3|Avg 5|MVP|Status Comment|Player Note\n")
    # Then write each player to file
    for each in json_stats['researchGridData']:
        #==============================================================================================
        # Note: Some players don't have Player Notes or Status Comments, which will throw an exception.
        # So this will default the value to N/A if there's no Player Notes or Status Comments
        player_note = each.get('player_note', "N/A")
        status_comment = each.get('status_comment', "N/A")
        #==============================================================================================
        line = (str(each['fn']) + '|' + str(each['ln']) + '|' + str(each['pos']) + '|' + str(each['pos2']) + '|' + str(each['team']) + '|' + str(each['tpts']) + '|' + str(each['rds']) + '|' +
            str(each['pts']) + '|' + str(each['avg']) + '|' + str(each['avg3']) + '|' + str(each['avg5']) + '|' + str(each['mvp']) + '|' +
            str(status_comment) + '|' +
            str(player_note) + '\n')
        print(line)
        file_output.write(line)
    file_output.close()


# Call the Download Stats function
download_stats()
```
