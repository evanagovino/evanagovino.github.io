---  
layout: post  
title: Custom NBA Trade Machine Part Four
---

Today I was able to expand the capability of the command-line program from two teams to four teams.

This took more time as I had anticipated, as I had hard-coded a lot of the code (<a href='http://www.evan.ag/NBATradeMachine3/' target="_blank">see here</a>) to only work for two teams, so I had to rewrite a number of functions to be able to take in a variant amount of teams, as well as add in the ability to point a player to different destinations.

Now the same functions for two teams work for four. You can propose a trade and then add/remove players from each team to make the trade work. I had some fun proposing a three-way Knicks/Cavs/Nuggets trade that would get Melo in Cleveland and Kyrie in Denver.

<img src="/../images/nbacommandline_screenshot7.png" width="800" />

There are still some kinks I need to work out. You can trade one player multiple times, and it doesn't allow you yet to change the destination for a given player once it's set (unless you remove that player and then add him back in).

I also need to work on adding more rules to the trades as discussed previously.

Overall this was a good lesson in making something scalable from the start, as I probably spent too much time working on something that should have been relatively easy. I still need to make a few adjustments to make it scale higher than four teams, but I'd guess that 90% of the work is done on that end.

Updated code is below.

```
import pandas as pd
import warnings
warnings.filterwarnings('ignore')

### SELECT TEAMS

def select_teams(new_df):
    team_list = []
    team_number = check_team_number()
    print('Who is your first team?')
    team_one  = check_team(new_df)
    team_list.append(team_one)
    print('Who is your second team?')
    team_two = check_team(new_df)
    team_list.append(team_two)
    if team_number > 2:
        print('Who is your third team?')
        team_three = check_team(new_df)
        team_list.append(team_three)
    else:
        team_three = None
    if team_number > 3:
        print('Who is your fourth team?')
        team_four = check_team(new_df)
        team_list.append(team_four)
    else:
        team_four = None
    return team_one, team_two, team_three, team_four, team_list

### START TRADING BLOCK

def start_trading_block(team, new_df, team_list):
    if team != None:
        trading_block = []
        trading_block = select_players(team, new_df, trading_block, team_list)
    else:
        trading_block = None
    return trading_block

def select_players(team, new_df, trading_block, team_list):
    z = 0
    while z < 1:
        player = check_player(team, new_df)
        if player != 'None':
            trading_block = add_player(player, team, trading_block, team_list)
        print('Do you want to add more players from ' + team + ' to the trading block?')
        answer = input()
        if answer == 'No':
            z += 1
    return trading_block

def add_player(player, team, which_team, team_list):
    add_this = new_df[(new_df['Team'] == team) & (new_df['Year'] == '2017-18') & (new_df['Player'] == player)]
    team_list_short = []
    for things in team_list:
        if things != team:
            team_list_short.append(things)
    if len(team_list_short) > 1:
        destination = check_team_in_list(team_list_short, player)
    else:
        destination = team_list_short[0]
    add_this['Destination'] = destination
    which_team.append(add_this)
    if isinstance(which_team, pd.DataFrame):
        print(which_team)
    else:
        print(pd.concat(which_team))
    return which_team

def check_team_in_list(team_list_short, player):
    z = 0
    while z == 0:
        print('Which team would you like ' + player + ' to be traded to?')
        answer = input()
        if answer in team_list_short:
            z += 1
    return answer

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

def check_team_number():
    new_team_number = 0
    while new_team_number == 0:
        print('How many teams will be in your trade? Pick between two and four.')
        my_number = input()
        try:
            if (int(my_number) > 1) and (int(my_number) < 5):
                new_team_number = int(my_number)
        except:
            continue
    return new_team_number


def check_dataframe(item):
    if isinstance(item, pd.DataFrame):
        pass
    else:
        item = pd.concat(item)
    return item

def process_trade(first_team, second_team, third_team, fourth_team, team_list):
    my_teams = [first_team, second_team, third_team, fourth_team]
    actual_teams = []
    for teams in my_teams:
        if teams != None:
            actual_teams.append(teams)
    dfs = []
    salaries = []
    team_names = []
    incoming_players = []
    incoming_salaries = []
    difference = []
    for i in range(len(actual_teams)):
        dfs.append(check_dataframe(actual_teams[i]))
        salaries.append(get_team_salary(dfs[i]))
        team_names.append(dfs[i]['Team'].mode()[0])
    all_incoming = get_all_incoming(dfs)
    for i in range(len(team_names)):
        incoming_team = get_incoming_salary(all_incoming, team_names[i])
        incoming_players.append(incoming_team)
        incoming_salary = get_team_salary(incoming_team)
        incoming_salaries.append(incoming_salary)

    for i in range(len(incoming_salaries)):
        team_difference = incoming_salaries[i] - salaries[i]
        difference.append(team_difference)

    greater = []
    for i in range(len(difference)):
        if abs(difference[i]) > 5000000:
            greater.append(i)
    print_out_incoming_outgoing(team_names, dfs, incoming_players, salaries, incoming_salaries)
    print()
    if len(greater) > 0:
        statements = []
        statement_list = "This trade didn't work. Please "
        statements.append(statement_list)
        for things in greater:
            minimum= abs(difference[things]) - 5000000
            statement_one = "subtract " + prettyify(minimum) + " from " + team_names[things] + "'s outgoing salary"
            statement_two = "add " + prettyify(minimum) + " to " + team_names[things] + "'s outgoing salary"
            if difference[things] > 0:
                statements.append(statement_two)
            else:
                statements.append(statement_one)
        final_statement = statements[0] + statements[1]
        i = 2
        while i < len(statements):
            final_statement = final_statement + ' and ' + statements[i]
            i += 1
        final_statement = final_statement + "."
        print(final_statement)
        print('Do you want to try and adjust the trade?')
        answer = input()
        if answer != 'No':
            for i in range(len(actual_teams)):
                actual_teams[i] = propose_adjustment(team_names[i], new_df, actual_teams[i], team_list)
                actual_teams[i] = propose_removal(team_names[i], actual_teams[i])
            i = 0
            new_teams = []
            #print(len(actual_teams))
            while i < 4:
                #print(i)
                if i < len(actual_teams):
                    new_teams.append(actual_teams[i])
                else:
                    new_teams.append(None)
                i += 1
            process_trade(new_teams[0],  new_teams[1], new_teams[2], new_teams[3], team_list)
    else:
        print('This trade works! Nice job!')
    

def print_out_incoming_outgoing(team_names, dfs, incoming_players, salaries, incoming_salaries):
    for i in range(len(team_names)):
        print(team_names[i] + "'s Outgoing Players:")
        print()
        print(dfs[i])
        print()
        print(team_names[i] + "'s Incoming Players:")
        print()
        print(incoming_players[i])
        print()
        print('Outgoing Salary:', team_names[i], prettyify(salaries[i]))
        print('Incoming Salary:', team_names[i], prettyify(incoming_salaries[i]))
        print()
        

def get_all_incoming(dfs):
    all = pd.concat(dfs)
    return all

def get_incoming_salary(all_incoming, team):
    #incoming_salary = all_incoming['Salary'][all_incoming['Destination'] == team].str.replace('$','').str.replace(',','').astype('int').sum()
    incoming_salary = all_incoming[all_incoming['Destination'] == team]
    return(incoming_salary)


def propose_adjustment(team_name, new_df, team, team_list):
    print('Do you want to add more players from ' + team_name + ' to the trading block?')
    answer = input()
    if answer == 'Yes':
        team = select_players(team_name, new_df, team, team_list)
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

def print_team(team, new_df):
    z = new_df[(new_df['Team'] == team) & (new_df['Year'] == '2017-18')]
    print(z)



new_df = pd.read_pickle(path='newdf')
team_one, team_two, team_three, team_four, team_list = select_teams(new_df)
team_one_trading_block = start_trading_block(team_one, new_df, team_list)
team_two_trading_block = start_trading_block(team_two, new_df, team_list)
team_three_trading_block = start_trading_block(team_three, new_df, team_list)
team_four_trading_block = start_trading_block(team_four, new_df, team_list)
process_trade(team_one_trading_block, team_two_trading_block, team_three_trading_block, team_four_trading_block, team_list)

```
