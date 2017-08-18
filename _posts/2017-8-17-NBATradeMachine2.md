---  
layout: post  
title: Custom NBA Trade Machine Part Two 
---

So I finished the first part of the project, which was pulling the information about player salaries. This was much easier than expected, thanks to Basketball Reference which has all of this information conveniently available on <a href='https://www.basketball-reference.com/contracts/players.html' target="_blank">one page</a>.

The code below pulls the table from this page with the very simple Pandas 'Read HTML' function.

```
def create_initial_df(html='https://www.basketball-reference.com/contracts/players.html'):
    my_table = pd.read_html(html)
    df = my_table[0]
    new_cols = []
    for things in df.columns:
        new_cols.append(things[1])
    df.columns = new_cols
    df = df.drop('Rk', axis=1)
    df = df[(df['Player'] != 'Salary') & (df['Player'] != 'Player')]
    return df
```

If you look on their page, you'll see that the table is set up so that there is one player on each row, with multiple years of contracts represented on each row. This won't work for me because there are some years that have contract clauses such as Player Option, Team Option, etc... I want to change the structure of the dataframe so that each year has the player and one year (i.e one player will have multiple rows)

```
def create_player_list(df):
    i = 0
    player_list = []
    while i < len(players):
        j = 2
        while j < 8:
            my_list = list([df.iloc[i, 0], df.iloc[i, 1], df.columns[j], df.iloc[i, j]])
            player_list.append(my_list)
            j += 1
        i += 1
    return player_list

def create_new_df(df, player_list):
    new_df = pd.DataFrame(player_list)
    new_df.columns = ['Player', 'Team', 'Year', 'Salary']
    new_df['Type'] = 'Regular'
    return new_df
```

Now I want to pull the page using Beautiful Soup to find the salaries that are tagged as Player Option, Team Option, etc.... I can find the respective CSS Selectors for these and pull all instances of the page. I can then use the 'find previous sibling' function to find the tag for the associated player. I'll keep a list of the players and salaries tagged with this and then change the tag for those players accordingly.

```
def pull_content(html='https://www.basketball-reference.com/contracts/players.html'):
    tags = ['.salary-pl', 'salary-et', '.salary-tm']
    labels = ['Player Option', 'Early Termination Option', 'Team Option']
    webpage = requests.get(html)
    content = webpage.content
    soup = BeautifulSoup(content)
    return soup

def change_tags(new_df, soup):
    for i in range(len(tags)):
        for label in soup.select(tags[i]):
            try:
                money = label.text.strip()
                player = label.find_previous_sibling(attrs={'data-stat':'player'}).text.strip()
                label = labels[i]
                new_df['Type'][(new_df['Player'] == player) & (new_df['Salary'] == money)] = label
            except:
                continue
    return new_df
```

Now I have a dataframe with the player, year, salary, and type of salary!

<img src="/../images/bballdataframe_1.png" width="800" />

Next I can work on setting up the trade logic and getting it to work on the command line. Enjoying this so far!!


