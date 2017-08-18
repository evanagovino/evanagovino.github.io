---  
layout: post  
title: Custom NBA Trade Machine Part Three 
---

Today I was able to get a basic function of the Trade Machine working at the command line.

First it asks you which teams you want to trade with at the command line and shows you the rosters of each team (loaded from the dataframe created in <a href='http://www.evan.ag/NBATradeMachine2/' target="_blank">Part Two</a>).

<img src="/../images/nba_commandline_screenshot1.png" width="800" />

It then asks you which players you want to add to the trading block and lets you do it, player by player.

<img src="/../images/nba_commandline_screenshot2.png" width="800" />

Once the trade is done, it evaluates it to see if it makes sense within CBA standards (essentially if the combined salaries of each side is within $5,000,000 of each other).

<img src="/../images/nba_commandline_screenshot3.png" width="800" />

If the trade isn't allowed, you can make adjustments to add or remove players from each trading block.

<img src="/../images/nba_commandline_screenshot4.png" width="800" />

The program won't let you add players to the trading block who don't exist or who are on the wrong team, and won't let you remove players from the trading block who aren't there.

<img src="/../images/nba_commandline_screenshot5.png" width="800" />

Once the trade is finalized, it'll give you the final breakdown.

<img src="/../images/nba_commandline_screenshot6.png" width="800" />

I believe the program works as far as adding/removing players from different teams and properly evaluating trades. I do want to keep testing edge cases to see what I need to work on, and of course I need to go back and clean up the code.

From there, my next steps are expanding the capability of this program to four teams (only two right now) and adding rules to the program regarding which players can be traded. There are a lot of specific quirks, such as that players who were recently traded can't be in a deal and players who recently signed can't be in a deal. I need to play around with the ESPN Trade Machine to figure out exaclty what I need to incorporate. I then need to find a place to get the information needed for these rules (such as dates when players signed or got traded - I believe the NBA SubReddit has this information). At the worst case, I can just manually input this information from ESPN (MAYBE BUT I HOPE NOT).

After that, I'll work on the frontend so that people can use this! I'm pushing the predictive end of it until later - I'd rather get this production ready and out first.

This is a blast so far. Looking forward to getting it out. Below is the code for the program.

```
import pandas as pd

def select_teams(new_df):
    print('Who is your first team?')
    team_one  = check_team(new_df)
    print('Who is your second team?')
    team_two = check_team(new_df)
    return team_one, team_two

def start_trading_block(team, new_df):
    trading_block = []
    trading_block = select_players(team, new_df, trading_block)
    return trading_block

def select_players(team, new_df, trading_block):
    z = 0
    while z < 1:
        player = check_player(team, new_df)
        if player != 'None':
            trading_block = add_player(player, team, trading_block)
        print('Do you want to add more players from ' + team + ' to the trading block?')
        answer = input()
        if answer == 'No':
            z += 1
    return trading_block

def remove_player(player, trading_block):
    new_trading_block = []
    for things in trading_block:
        if things['Player'].mode()[0] != player:
            new_trading_block.append(things)
    return new_trading_block

def remove_players(team, trading_block):
    z = 0
    while z < 1:
        player = check_player_trading_block(team,trading_block)
        if player != 'None':
            trading_block = remove_player(player, trading_block)
            print(pd.concat(trading_block))
        print('Do you want to remove more players from ' + team + "'s trading block?")
        answer = input()
        if answer == 'No':
            z += 1
    return trading_block


def check_player_trading_block(team, new_df):
    remove_player = ''
    new_df = pd.concat(new_df)
    while len(remove_player) == 0:
        print(new_df)
        print('Which players from the ' + team + "'s trading block do you want to remove?")
        player = input()
        if (player in new_df['Player'].unique()) or (player == 'None'):
            remove_player = player
        else:
            print("Sorry, that player's not on the trading block.")
            print("Please pick a player on the trading block.")
    return remove_player

def check_player(team, new_df):
    new_player = ''
    while len(new_player) == 0:
        print_team(team, new_df)
        print('Which players from ' + team + ' do you want to trade?')
        player = input()
        if (player in new_df['Player'][new_df['Team'] == team].unique()) or (player == 'None'):
            new_player = player
        else:
            print("Sorry, that's not the name of a player on " + team)
            print("Please pick a player on " + team + " .")
    return new_player


def check_team(new_df):
    new_team = ''
    while len(new_team) == 0:
        team = input()
        if team in new_df['Team'].unique():
            new_team = team
        else:
            print("Sorry, that's not the name of a team.")
            print('Please pick a team.')
    return new_team

def check_dataframe(item):
    if isinstance(item, pd.DataFrame):
        pass
    else:
        item = pd.concat(item)
    return item

def process_trade(first_team, second_team):
    first_team_df = check_dataframe(first_team)
    second_team_df = check_dataframe(second_team)
    first_team_salary = get_team_salary(first_team_df)
    second_team_salary = get_team_salary(second_team_df)
    first_team_name = first_team_df['Team'].mode()[0]
    second_team_name = second_team_df['Team'].mode()[0]
    difference = first_team_salary - second_team_salary
    print(first_team_name + ' outgoing Players:')
    print()
    print(first_team_df)
    print()
    print(second_team_name + ' outgoing Players:')
    print()
    print(second_team_df)
    print()
    print('Outgoing Salary:', first_team_name, prettyify(first_team_salary))
    print('Outgoing Salary:', second_team_name, prettyify(second_team_salary))
    print()
    if abs(difference) > 5000000:
        minimum= abs(difference) - 5000000
        statement = "This trade didn't work. Please add " + prettyify(minimum) + " to "
        statement_three = "'s outgoing salary or subtract " + prettyify(minimum) + " from "
        statement_five = "'s outgoing salary."
        if difference > 0:
            statement_two = second_team_name
            statement_four = first_team_name
        else:
            statement_two = first_team_name
            statement_four = second_team_name
        print(statement + statement_two + statement_three + statement_four + statement_five)
        print('Do you want to try and adjust the trade?')
        answer = input()
        if answer != 'No':
            first_team = propose_adjustment(first_team_name, new_df, first_team)
            second_team = propose_adjustment(second_team_name,new_df, second_team)
            first_team = propose_removal(first_team_name, first_team)
            second_team = propose_removal(second_team_name, second_team)
            process_trade(first_team, second_team)
    else:
        print('This trade works! Nice job!')
        first_team_receiving = print_out_players(second_team_df['Player'])
        second_team_receiving = print_out_players(first_team_df['Player'])
        print(first_team_name + ' gets: ' + first_team_receiving)
        print(second_team_name + ' gets: ' + second_team_receiving)

def propose_adjustment(team_name, new_df, team):
    print('Do you want to add more players from ' + team_name + ' to the trading block?')
    answer = input()
    if answer == 'Yes':
        team = select_players(team_name, new_df, team)
    return team

def propose_removal(team_name, trading_block):
    print('Do you want to remove players on ' + team_name + ' from the trading block?')
    answer = input()
    if answer == 'Yes':
        trading_block = remove_players(team_name, trading_block)
    return trading_block

def print_out_players(series):
    receiving = ''
    for things in series:
        receiving = receiving + things + ' '
    return receiving

def get_team_salary(team):
    salary = team['Salary'].str.replace('$','').str.replace(',','').astype('int').sum()
    return(salary)

def create_tax_statement(difference, cap_room):
    if cap_room < 0:
        pretty = prettyify(abs(difference))
        if difference > 0:
            statement = 'This team is ' + pretty + ' below the tax curtain.'
        else:
            statement = 'This team is ' + pretty + ' above the tax curtain.'
    else:
        statement = ''
    return statement

def create_minimum_statement(difference):
    pretty = prettyify(abs(difference))
    if difference > 0:
        statement = 'The team is ' + pretty + ' below the minimum salary limit.'
    else:
        statement = ''
    return statement

def prettyify(value):
    pretty = '${:,.2f}'.format(value)
    return pretty

def cap_room_statement(difference):
    pretty = prettyify(abs(difference))
    if difference > 0:
        statement = 'The team is ' + pretty + ' below the cap.'
    else:
        statement = 'The team is ' + pretty + ' above the cap.'
    return statement

def get_team_total(team, cap_figure=9909300, minimum_figure=89184000, tax_figure=119266000):
    total = get_team_salary(team)
    total_pretty = prettyify(total)
    cap_room = cap_figure - total
    minimum_difference = minimum_figure - total
    tax_figure = tax_figure - total
    statement = cap_room_statement(cap_room)
    minimum_statement = create_minimum_statement(minimum_difference)
    tax_statement = create_tax_statement(tax_figure, cap_room)
    print('The total salary is ' + total_pretty + '.')
    print(statement)
    if len(minimum_statement) > 0:
        print(minimum_statement)
    if len(tax_statement) > 0:
        print(tax_statement)

def add_player(player, team, which_team):
    add_this = new_df[(new_df['Team'] == team) & (new_df['Year'] == '2017-18') & (new_df['Player'] == player)]
    which_team.append(add_this)
    if isinstance(which_team, pd.DataFrame):
        print(which_team)
    else:
        print(pd.concat(which_team))
    return which_team

def print_team(team, new_df):
    z = new_df[(new_df['Team'] == team) & (new_df['Year'] == '2017-18')]
    print(z)



new_df = pd.read_pickle(path='newdf')
team_one, team_two = select_teams(new_df)
team_one_trading_block = start_trading_block(team_one, new_df)
team_two_trading_block = start_trading_block(team_two, new_df)
process_trade(team_one_trading_block, team_two_trading_block)
```



