# FOOTBALL-MANAGEMENT-SYSTEM-WITH-SQL-AND-PYTHON 
# the code is performed on python (jupyter notebook)



import pyodbc
import pandas as pd
import matplotlib.pyplot as plt

# Define the connection string for Windows Authentication
server = 'LAPTOP-7VQEVQPI\\SQLEXPRESS'  # Use double backslashes here
database = 'FootballManagement1'
connection_string = f'DRIVER={{ODBC Driver 17 for SQL Server}};SERVER={server};DATABASE={database};Trusted_Connection=yes;'

# Connect to the database
try:
    conn = pyodbc.connect(connection_string)
    cursor = conn.cursor()
    print("Connection successful!")
except pyodbc.Error as ex:
    sqlstate = ex.args[1]
    print(f"Connection failed: {sqlstate}")

def display_dataframe(df):
    print(df)

def display_players():
    cursor.execute("SELECT * FROM Player")
    rows = cursor.fetchall()
    df = pd.DataFrame([tuple(row) for row in rows], columns=[desc[0] for desc in cursor.description])
    display_dataframe(df)

def display_teams():
    cursor.execute("SELECT * FROM Team")
    rows = cursor.fetchall()
    df = pd.DataFrame([tuple(row) for row in rows], columns=[desc[0] for desc in cursor.description])
    display_dataframe(df)

def display_matches():
    cursor.execute("SELECT * FROM Match")
    rows = cursor.fetchall()
    df = pd.DataFrame([tuple(row) for row in rows], columns=[desc[0] for desc in cursor.description])
    display_dataframe(df)

def display_player_stats(match_id):
    query = """
    SELECT p.FirstName, p.LastName, ps.Goals, ps.Assists, ps.YellowCards, ps.RedCards
    FROM PlayerStats ps
    JOIN Player p ON ps.PlayerID = p.PlayerID
    WHERE ps.MatchID = ?
    """
    cursor.execute(query, match_id)
    rows = cursor.fetchall()
    df = pd.DataFrame([tuple(row) for row in rows], columns=[desc[0] for desc in cursor.description])
    display_dataframe(df)

def display_tournament_matches():
    query = """
    SELECT t.TournamentName, m.MatchDate, ht.TeamName AS HomeTeam, at.TeamName AS AwayTeam
    FROM TournamentMatches tm
    JOIN Tournament t ON tm.TournamentID = t.TournamentID
    JOIN Match m ON tm.MatchID = m.MatchID
    JOIN Team ht ON m.HomeTeamID = ht.TeamID
    JOIN Team at ON m.AwayTeamID = at.TeamID
    """
    cursor.execute(query)
    rows = cursor.fetchall()
    df = pd.DataFrame([tuple(row) for row in rows], columns=[desc[0] for desc in cursor.description])
    display_dataframe(df)

def display_match_stadiums():
    query = """
    SELECT m.MatchDate, s.StadiumName, s.Location, s.Capacity
    FROM MatchStadium ms
    JOIN Match m ON ms.MatchID = m.MatchID
    JOIN Stadium s ON ms.StadiumID = s.StadiumID
    """
    cursor.execute(query)
    rows = cursor.fetchall()
    df = pd.DataFrame([tuple(row) for row in rows], columns=[desc[0] for desc in cursor.description])
    display_dataframe(df)

def display_ticket_sales(match_id):
    query = """
    SELECT ts.FanName, ts.SeatNumber, ts.Price
    FROM TicketSales ts
    WHERE ts.MatchID = ?
    """
    cursor.execute(query, match_id)
    rows = cursor.fetchall()
    df = pd.DataFrame([tuple(row) for row in rows], columns=[desc[0] for desc in cursor.description])
    display_dataframe(df)

def display_all_tables():
    cursor.execute("SELECT TABLE_NAME FROM INFORMATION_SCHEMA.TABLES WHERE TABLE_TYPE = 'BASE TABLE' AND TABLE_CATALOG='FootballManagement'")
    rows = cursor.fetchall()
    df = pd.DataFrame([tuple(row) for row in rows], columns=['TableName'])
    display_dataframe(df)

def add_player(first_name, last_name, dob, position, nationality):
    query = "INSERT INTO Player (FirstName, LastName, DateOfBirth, Position, Nationality) VALUES (?, ?, ?, ?, ?)"
    cursor.execute(query, (first_name, last_name, dob, position, nationality))
    conn.commit()
    print(f'Player {first_name} {last_name} added.')

def add_team(team_name, coach_name):
    query = "INSERT INTO Team (TeamName, CoachName) VALUES (?, ?)"
    cursor.execute(query, (team_name, coach_name))
    conn.commit()
    print(f'Team {team_name} added.')

def add_match(home_team_id, away_team_id, match_date, home_team_score=None, away_team_score=None):
    query = "INSERT INTO Match (HomeTeamID, AwayTeamID, MatchDate, HomeTeamScore, AwayTeamScore) VALUES (?, ?, ?, ?, ?)"
    cursor.execute(query, (home_team_id, away_team_id, match_date, home_team_score, away_team_score))
    conn.commit()
    print(f'Match added between Home Team ID {home_team_id} and Away Team ID {away_team_id}.')

def delete_player(player_id):
    query = "DELETE FROM Player WHERE PlayerID = ?"
    cursor.execute(query, player_id)
    conn.commit()
    print(f'Player with ID {player_id} deleted.')

def delete_team(team_id):
    query = "DELETE FROM Team WHERE TeamID = ?"
    cursor.execute(query, team_id)
    conn.commit()
    print(f'Team with ID {team_id} deleted.')

def delete_match(match_id):
    query = "DELETE FROM Match WHERE MatchID = ?"
    cursor.execute(query, match_id)
    conn.commit()
    print(f'Match with ID {match_id} deleted.')

def update_player(player_id, first_name=None, last_name=None, dob=None, position=None, nationality=None):
    query = "UPDATE Player SET "
    params = []
    
    if first_name:
        query += "FirstName = ?, "
        params.append(first_name)
    if last_name:
        query += "LastName = ?, "
        params.append(last_name)
    if dob:
        query += "DateOfBirth = ?, "
        params.append(dob)
    if position:
        query += "Position = ?, "
        params.append(position)
    if nationality:
        query += "Nationality = ? "
        params.append(nationality)
    
    if query.endswith(', '):
        query = query[:-2]
    
    query += " WHERE PlayerID = ?"
    params.append(player_id)
    
    cursor.execute(query, params)
    conn.commit()
    print(f'Player with ID {player_id} updated.')

def update_match_score(match_id, home_team_score, away_team_score):
    query = "UPDATE Match SET HomeTeamScore = ?, AwayTeamScore = ? WHERE MatchID = ?"
    cursor.execute(query, (home_team_score, away_team_score, match_id))
    conn.commit()
    print(f'Match with ID {match_id} updated.')

def plot_player_stats():
    cursor.execute("SELECT p.FirstName, p.LastName, ps.Goals, ps.Assists FROM PlayerStats ps JOIN Player p ON ps.PlayerID = p.PlayerID")
    rows = cursor.fetchall()
    df = pd.DataFrame([tuple(row) for row in rows], columns=[desc[0] for desc in cursor.description])
    
    fig, ax = plt.subplots(2, 1, figsize=(10, 8))
    
    df.groupby(['FirstName', 'LastName'])['Goals'].sum().plot(kind='bar', ax=ax[0])
    ax[0].set_title('Total Goals by Player')
    ax[0].set_ylabel('Goals')
    
    df.groupby(['FirstName', 'LastName'])['Assists'].sum().plot(kind='bar', ax=ax[1])
    ax[1].set_title('Total Assists by Player')
    ax[1].set_ylabel('Assists')
    
    plt.tight_layout()
    plt.show()

def main():
    while True:
        choice = input("\nDo you need to make any changes or view any data? (yes/no): ").strip().lower()
        if choice == 'no':
            print("Exiting...")
            break

        print("\n--- Football Management System ---")
        print("1. Display Data")
        print("2. Update Data")
        print("3. Delete Data")
        print("4. Plot Player Statistics")
        print("5. Exit")
        choice = input("Enter your choice: ")
        
        if choice == '1':
            print("\n--- Display Data ---")
            print("1. Display Players")
            print("2. Display Teams")
            print("3. Display Matches")
            print("4. Display Player Stats for a Match")
            print("5. Display Tournament Matches")
            print("6. Display Match Stadiums")
            print("7. Display Ticket Sales for a Match")
            print("8. Display All Tables")
            display_choice = input("Enter your choice: ")
            
            if display_choice == '1':
                display_players()
            elif display_choice == '2':
                display_teams()
            elif display_choice == '3':
                display_matches()
            elif display_choice == '4':
                match_id = input("Enter match ID: ")
                display_player_stats(match_id)
            elif display_choice == '5':
                display_tournament_matches()
            elif display_choice == '6':
                display_match_stadiums()
            elif display_choice == '7':
                match_id = input("Enter match ID: ")
                display_ticket_sales(match_id)
            elif display_choice == '8':
                display_all_tables()
            else:
                print("Invalid choice. Please try again.")
        
        elif choice == '2':
            print("\n--- Update Data ---")
            print("1. Update Player")
            print("2. Update Match Score")
            update_choice = input("Enter your choice: ")
            
            if update_choice == '1':
                player_id = input("Enter player ID to update: ")
                first_name = input("Enter first name (optional): ")
                last_name = input("Enter last name (optional): ")
                dob = input("Enter date of birth (optional) (YYYY-MM-DD): ")
                position = input("Enter position (optional): ")
                nationality = input("Enter nationality (optional): ")
                update_player(player_id, first_name or None, last_name or None, dob or None, position or None, nationality or None)
            elif update_choice == '2':
                match_id = input("Enter match ID to update: ")
                home_team_score = input("Enter home team score: ")
                away_team_score = input("Enter away team score: ")
                update_match_score(match_id, home_team_score, away_team_score)
            else:
                print("Invalid choice. Please try again.")
        
        elif choice == '3':
            print("\n--- Delete Data ---")
            print("1. Delete Player")
            print("2. Delete Team")
            print("3. Delete Match")
            delete_choice = input("Enter your choice: ")
            
            if delete_choice == '1':
                player_id = input("Enter player ID to delete: ")
                delete_player(player_id)
            elif delete_choice == '2':
                team_id = input("Enter team ID to delete: ")
                delete_team(team_id)
            elif delete_choice == '3':
                match_id = input("Enter match ID to delete: ")
                delete_match(match_id)
            else:
                print("Invalid choice. Please try again.")
        
        elif choice == '4':
            plot_player_stats()
        
        elif choice == '5':
            print("Exiting...")
            break
        
        else:
            print("Invalid choice. Please try again.")

# Run the main function
if __name__ == "__main__":
    main()

# Close the connection
conn.close()
