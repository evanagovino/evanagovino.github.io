---  
layout: post  
title: NBA Projections Part One
---

Unfortunately I haven't had much time to keep up with this project in the past few weeks, but with the NBA season around the corner, I wanted to make team projections for the season!

My idea was that I could combine this with the Trade Machine idea by projecting the performance of individual players (via Win Shares) and then aggregating the performance of the aggregate players to make a team projection of wins. This way I could use the individual projections for players for the trade machine.

Win Shares are a semi-convenient way to be able to do that. I say 'semi' because the aggregated Win Shares for a team are typically closer to a team's 'Pythagorean' wins rather than their actual wins. The 'Pythagorean' formula tracks expected win-loss values for a team based on how many points they've scored throughout the season, and how many points have been scored about them. In essence, it rewards teams for blowing out other teams and having close losses, and penalizes them for getting blown out while having close wins. Shoutout to Don La Greca!

<iframe width="560" height="315" src="https://www.youtube.com/embed/CVoiq6CadrY" frameborder="0" allowfullscreen></iframe>

I pulled the aggregate win shares and team performances for each team for the past five years. The aggregate win shares show a total deviation from the actual wins by 358 games, a little less than 2.4 wins per team (30 teams x 5 years = 150). They deviate from the Pythagorean wins by 151 wins, which is less than 1 win per team!

Still, I'm more concerned with predicting the teams' actual records. Below is the distribution of aggregate wins versus actual wins, which is pretty close to a normal distribution.

<img src="/../images/winshares_v_wins.png" width="800" />

<img src="/../images/winshares_v_wins_two.png" width="800" />

43% of teams fall within an error of one game, 62% of teams fall within an error of two games and 74% of teams fall within an error of three games. This is far from perfect, but it shows that aggregate win shares can decently predict where a team will end up.

Of course that depends on me actually predicting player win shares accurately. So that's my next step! How can I predict an individual player's win shares, taking into account aging, injuries and rookies with no prior history in the league? (I'll ignore trades since those are impossible to predict.)

Looking forward to the next steps! Below is the code I used to pull the aggregate win shares for every team from the past five years.

```
import pandas as pd
from bs4 import BeautifulSoup
import requests
import re
import seaborn as sns

def build_team_list(year):
	html = 'https://www.basketball-reference.com/leagues/NBA_' + str(year) + '.html'
	webpage = requests.get(html)
	content = webpage.content
	soup = BeautifulSoup(re.sub("<!--|-->","", content.decode('utf-8')),'lxml')
	l = soup.select('#div_team-stats-per_game')[0]
	teams = l.find_all('a')
	my_teams = []
	for things in teams:
		team_list = str(things).split('/')
		my_url = team_list[2] + '/' + str(year) + '.html'
		my_teams.append(my_url)
	return(my_teams)

def get_record(p, term, spot):
	record = 'DIDNT WORK'
	for i in range(len(p)):
		text_split = p[i].text.split(' ')
		if (text_split[0] == term) or (text_split[0] == '\n' + term):
			record = text_split[spot][:2]
			break
	return record


def get_table(team):
	url = 'https://www.basketball-reference.com/teams/' + team
	webpage = requests.get(url)
	content = webpage.content
	cleaned_soup = BeautifulSoup(re.sub("<!--|-->","", content.decode('utf-8')),'lxml')
	l = cleaned_soup.find_all('table', {'id': 'advanced'})[0]
	header = cleaned_soup.select('h1')
	text = header[0].text
	p = cleaned_soup.select('#meta p')
	actual_record = get_record(p, 'Record:', spot=1)
	expected_record = get_record(p, 'Expected', spot=2)
	team_name = text.split('\n')[2]
	return l, actual_record, expected_record, team_name


def get_aggregate_score(table):
	d = []
	row_marker = 0
	for row in table.find_all('tr'):
	    column_marker = 0
	    columns = row.find_all('td')
	    l = []
	    for column in columns:
	        l.append(column.get_text())
	        column_marker += 1
	    d.append(l)
	    row_marker += 1
	i = 1
	ws = 0
	wshares_rounded = 0
	while i < len(d):
		try:
			wsp48 = float(d[i][20])
		except:
			wsp48 = 0
		try:
			minutes = float(d[i][3])
		except:
			minutes = 0
		try:
			ws_rounded = float(d[i][19])
		except:
			ws_rounded = 0
		games = minutes / 48
		win_shares = wsp48 * games 
		ws = ws + win_shares
		wshares_rounded = wshares_rounded + ws_rounded
		i += 1
	return(ws, wshares_rounded)

def get_all_years(min_year, max_year):
	year = max_year
	list_of_dfs = []
	while year > min_year:
		my_teams = build_team_list(year)
		df = pd.DataFrame(columns=['team', 'record', 'expected_record','ws','ws_rounded','year'])
		for i in range(len(my_teams)):
			print(my_teams[i])
			table, actual_record, expected_record, team_name = get_table(my_teams[i])
			ws, ws_rounded = get_aggregate_score(table)
			df.loc[i] = [team_name, actual_record, expected_record, ws, ws_rounded, year]
		list_of_dfs.append(df)
		year = year - 1
	total_df = pd.concat(list_of_dfs)
	return(total_df)


total_df = get_all_years(2012,2017)
total_df.to_pickle('total_df.pkl')
```











