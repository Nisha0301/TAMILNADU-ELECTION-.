# TAMILNADU-ELECTION-.
import sqlite3
import csv
import smtplib
from email.mime.text import MIMEText

# Database setup
def create_database():
    conn = sqlite3.connect('tamilnadu_election.db')
    c = conn.cursor()
    c.execute('''CREATE TABLE IF NOT EXISTS voters 
                 (id INTEGER PRIMARY KEY, name TEXT, email TEXT, team TEXT)''')
    conn.commit()
    conn.close()

def insert_voter(name, email, team):
    conn = sqlite3.connect('tamilnadu_election.db')
    c = conn.cursor()
    c.execute("INSERT INTO voters (name, email, team) VALUES (?, ?, ?)", (name, email, team))
    conn.commit()
    conn.close()

def get_voters():
    conn = sqlite3.connect('tamilnadu_election.db')
    c = conn.cursor()
    c.execute("SELECT * FROM voters")
    voters = c.fetchall()
    conn.close()
    return voters

def search_voter(name):
    conn = sqlite3.connect('tamilnadu_election.db')
    c = conn.cursor()
    c.execute("SELECT * FROM voters WHERE name=?", (name,))
    voter = c.fetchone()
    conn.close()
    return voter

def count_votes():
    conn = sqlite3.connect('tamilnadu_election.db')
    c = conn.cursor()
    c.execute("SELECT team, COUNT(*) FROM voters GROUP BY team")
    vote_counts = c.fetchall()
    conn.close()
    return vote_counts

# File handling
def save_voters_to_file():
    voters = get_voters()
    with open('voters.csv', 'w', newline='') as file:
        writer = csv.writer(file)
        writer.writerow(['ID', 'Name', 'Email', 'Team'])
        writer.writerows(voters)

# Sending email
def send_thank_you_emails():
    voters = get_voters()
    for voter in voters:
        send_email(voter[2])

def send_email(email):
    msg = MIMEText("Thank you for voting!")
    msg['Subject'] = 'Tamilnadu Election'
    msg['From'] = 'your_email@example.com'
    msg['To'] = email

    try:
        with smtplib.SMTP('smtp.example.com', 587) as server:
            server.starttls()
            server.login('your_email@example.com', 'your_password')
            server.send_message(msg)
            print(f"Email sent to {email}")
    except Exception as e:
        print(f"Failed to send email to {email}: {e}")

# Main program
def main():
    create_database()
    
    while True:
        print("\nTamilnadu Election System")
        print("1. Add Voter")
        print("2. Search Voter")
        print("3. Count Votes")
        print("4. Save Voters to File")
        print("5. Send Thank You Emails")
        print("6. Exit")
        
        choice = input("Enter your choice: ")
        
        if choice == '1':
            name = input("Enter name: ")
            email = input("Enter email: ")
            team = input("Enter team: ")
            insert_voter(name, email, team)
        elif choice == '2':
            name = input("Enter name to search: ")
            voter = search_voter(name)
            if voter:
                print(f"Voter Found: ID={voter[0]}, Name={voter[1]}, Email={voter[2]}, Team={voter[3]}")
            else:
                print("Voter not found")
        elif choice == '3':
            vote_counts = count_votes()
            for team, count in vote_counts:
                print(f"Team: {team}, Votes: {count}")
        elif choice == '4':
            save_voters_to_file()
            print("Voters saved to file")
        elif choice == '5':
            send_thank_you_emails()
        elif choice == '6':
            break
        else:
            print("Invalid choice. Please try again.")

if __name__ == "__main__":
    main()
