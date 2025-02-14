import datetime
import webbrowser
import smtplib
import os
import traceback
import msal
import requests

def take_command():
    command = input("Enter your command: ").lower()
    return command

def respond(text):
    print(text)

def send_email_via_graph_api(to, subject, body):
    try:
        client_id = 'your_client_id'
        client_secret = 'your_client_secret'
        tenant_id = 'your_tenant_id'

        authority = f"https://login.microsoftonline.com/{tenant_id}"
        app = msal.ConfidentialClientApplication(client_id, authority=authority, client_credential=client_secret)

        scope = ["https://graph.microsoft.com/.default"]
        result = app.acquire_token_silent(scope, account=None)

        if not result:
            result = app.acquire_token_for_client(scopes=scope)

        if "access_token" in result:
            headers = {
                'Authorization': 'Bearer ' + result['access_token'],
                'Content-Type': 'application/json'
            }

            email_data = {
                "message": {
                    "subject": subject,
                    "body": {
                        "contentType": "Text",
                        "content": body
                    },
                    "toRecipients": [
                        {
                            "emailAddress": {
                                "address": to
                            }
                        }
                    ]
                },
                "saveToSentItems": "true"
            }

            response = requests.post(
                'https://graph.microsoft.com/v1.0/me/sendMail',
                headers=headers,
                json=email_data
            )

            if response.status_code == 202:
                respond("Email sent successfully via Microsoft Graph API!")
            else:
                respond(f"Failed to send email. Error: {response.status_code} {response.text}")
        else:
            respond("Failed to acquire token.")
    except Exception as e:
        respond(f"Failed to send email. Error: {str(e)}")
        traceback.print_exc()

def calculate(expression):
    try:
        result = eval(expression)
        respond(f"The result is: {result}")
    except Exception as e:
        respond(f"Failed to calculate. Error: {str(e)}")
        traceback.print_exc()

def main():
    while True:
        command = take_command()

        if 'time' in command:
            now = datetime.datetime.now().strftime("%H:%M:%S")
            respond(f"The current time is {now}")

        elif 'open website' in command:
            respond("Please provide the URL of the website:")
            url = input("Enter URL: ")
            try:
                webbrowser.open(url)
                respond(f"Opening {url}")
            except Exception as e:
                respond(f"Failed to open website. Error: {str(e)}")
                traceback.print_exc()

        elif 'search google' in command:
            respond("What do you want to search for on Google?")
            query = input("Enter search query: ")
            url = f"https://www.google.com/search?q={query}"
            try:
                webbrowser.open(url)
                respond(f"Searching for {query} on Google")
            except Exception as e:
                respond(f"Failed to search on Google. Error: {str(e)}")
                traceback.print_exc()

        elif 'send email' in command:
            respond("Please provide the recipient email address:")
            to = input("Enter recipient email: ")
            respond("Please provide the subject of the email:")
            subject = input("Enter email subject: ")
            respond("Please provide the body of the email:")
            body = input("Enter email body: ")
            send_email_via_graph_api(to, subject, body)

        elif 'calculate' in command:
            respond("Please provide the mathematical expression:")
            expression = input("Enter expression: ")
            calculate(expression)

        elif 'exit' in command:
            respond("Goodbye!")
            break

        else:
            respond("Sorry, I didn't understand that command.")

if __name__ == "__main__":
       main()
