

import sys
import requests
from bs4 import BeautifulSoup
import urllib.parse

# ANSI color codes for terminal output
RED = "\033[31m"
GREEN = "\033[32m"
YELLOW = "\033[33m"
BLUE = "\033[34m"
RESET = "\033[0m"

def search_ios_app(app_name):
    query = urllib.parse.quote(app_name)
    url = f"https://itunes.apple.com/search?term={query}&entity=software"
    try:
        response = requests.get(url)
        response.raise_for_status()
    except Exception as e:
        sys.exit(f"{RED}[iOS] Error fetching app info: {e}{RESET}")

    results = response.json().get("results", [])
    if not results:
        sys.exit(f"{RED}[iOS] App '{app_name}' not found. Exiting.{RESET}")
    
    app = results[0]  # take the first match
    name = app.get("trackName", "Unknown App")
    version = app.get("version", "N/A")
    release_notes = app.get("releaseNotes", "No release notes provided.")
    print(f"\n{BLUE}[iOS]{RESET} {GREEN}{name}{RESET}")
    print(f"{YELLOW}Version:{RESET} {version}")
    print(f"{YELLOW}Release Notes:{RESET}")
    print(release_notes)

def search_android_app(app_name):
    query = urllib.parse.quote(app_name)
    search_url = f"https://play.google.com/store/search?q={query}&c=apps&hl=en&gl=us"
    headers = {"User-Agent": "Mozilla/5.0"}
    
    try:
        r = requests.get(search_url, headers=headers)
        r.raise_for_status()
        soup = BeautifulSoup(r.text, "html.parser")
        app_link = soup.select_one("a[href^='/store/apps/details?id=']")
        if not app_link:
            sys.exit(f"{RED}[Android] App '{app_name}' not found on Google Play. Exiting.{RESET}")
        
        app_url = "https://play.google.com" + app_link["href"]
        r = requests.get(app_url, headers=headers)
        r.raise_for_status()
        soup = BeautifulSoup(r.text, "html.parser")
        
        title_tag = soup.find("h1", class_="AHFaub")
        title = title_tag.text.strip() if title_tag else "Unknown App"
        whats_new_section = soup.find("div", class_="W4P4ne")
        release_notes = whats_new_section.text.strip() if whats_new_section else "No release notes found."
        
        print(f"\n{BLUE}[Android]{RESET} {GREEN}{title}{RESET}")
        print(f"{YELLOW}Release Notes:{RESET}")
        print(release_notes)
    except Exception as e:
        sys.exit(f"{RED}[Android] Error while fetching app details: {e}{RESET}")

def main():
    app_name = input(f"{YELLOW}Enter the app name to check: {RESET}").strip()
    if not app_name:
        sys.exit(f"{RED}No app name provided. Exiting.{RESET}")
    
    print(f"\n{BLUE}Fetching latest release info for '{app_name}'...{RESET}")
    search_ios_app(app_name)
    search_android_app(app_name)

if __name__ == "__main__":
    main()

How It Works
	•	Colored Output:
	•	Errors are printed in red.
	•	Labels such as [iOS] and [Android] appear in blue.
	•	App names are shown in green, while version labels are yellow.
	•	Error Handling:
	•	The script checks if an app name is provided and exits if not.
	•	If an app isn’t found on either store or if there’s an HTTP error, it prints an error message and exits.

Running the Script
	1.	Save the script as app_release_tracker.py.
	2.	Install the required packages if you haven’t already:

pip3 install requests beautifulsoup4


	3.	Run the script:

python3 app_release_tracker.py


	4.	When prompted, enter the app name (for example, nba).

This version should meet your needs by providing a colored, interactive experience and exiting with an error if any essential part is missing.
