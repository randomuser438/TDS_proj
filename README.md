**I.An explanation of how you scraped the data**

Following code was used to scarpe the data 
'''
import requests
import csv


GITHUB_API_URL = "https://api.github.com"
TOKEN = "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"
HEADERS = {
    "Authorization": f"token {TOKEN}"
}


def get_users_from_bangalore():
    users = []
    page = 1
    while True:
        response = requests.get(f"{GITHUB_API_URL}/search/users?q=location:bangalore+followers:>100&per_page=100&page={page}", headers=HEADERS)
        data = response.json()
        users.extend(data['items'])
        if 'next' not in response.links:
            break
        page += 1
    return users

def get_user_details(username):
    response = requests.get(f"{GITHUB_API_URL}/users/{username}", headers=HEADERS)
    user_data = response.json()
    company = user_data.get('company', '')
    if company:
        company = company.strip().lstrip('@').upper()
    return {
        'login': user_data.get('login', ''),
        'name': user_data.get('name', ''),
        'company': company,
        'location': user_data.get('location', ''),
        'email': user_data.get('email', ''),
        'hireable': user_data.get('hireable', ''),
        'bio': user_data.get('bio', ''),
        'public_repos': user_data.get('public_repos', 0)
    }

def write_to_csv(users):
    with open('users.csv', mode='w', newline='', encoding='utf-8') as file:
        writer = csv.DictWriter(file, fieldnames=['login', 'name', 'company', 'location', 'email', 'hireable', 'bio', 'public_repos'])
        writer.writeheader()
        for user in users:
            writer.writerow(user)

def write_repositories_to_csv(repositories):
    with open('repositories.csv', mode='w', newline='', encoding='utf-8') as file:
        writer = csv.DictWriter(file, fieldnames=['login', 'full_name', 'created_at', 'stargazers_count', 'watchers_count', 'language'])
        writer.writeheader()
        for repo in repositories:
            writer.writerow(repo)

def get_user_repositories(username):
    repositories = []
    page = 1
    while True:
        response = requests.get(f"{GITHUB_API_URL}/users/{username}/repos?sort=pushed&per_page=100&page={page}", headers=HEADERS)
        data = response.json()
        repositories.extend(data)
        if len(data) < 100 or len(repositories) >= 500:
            break
        page += 1
    return repositories[:500]


def main():
    users = get_users_from_bangalore()
    user_details = [get_user_details(user['login']) for user in users]
    write_to_csv(user_details)
    all_repositories = []
    for user in user_details:
        repos = get_user_repositories(user['login'])
        for repo in repos:
            all_repositories.append({
                'login': user['login'],
                'full_name': repo.get('full_name', ''),
                'created_at': repo.get('created_at', ''),
                'stargazers_count': repo.get('stargazers_count', 0),
                'watchers_count': repo.get('watchers_count', 0),
                'language': repo.get('language', '')
            })
    write_repositories_to_csv(all_repositories)



if __name__ == "__main__":
    main()
    '''
Where the used functions perfrom the following functions:
1.Constants and Headers Setup:
GITHUB_API_URL is the base URL for GitHub's API.
TOKEN holds the GitHub API token for authentication.
HEADERS dictionary is used to include the authorization header with each API request.
Functions:

2.get_users_from_bangalore:
Fetches users from Bangalore who have more than 100 followers.
The GitHub API request is paginated to get multiple pages of results (100 users per page).
For each page, it retrieves the list of users and adds them to the users list.
It breaks the loop when there are no more pages ('next' link is absent in the response.links).

3.get_user_details:
Takes a GitHub username and fetches detailed information about that user.
The function retrieves details such as login, name, company, location, email, hireable status, bio, and public_repos count.
It returns these details in a dictionary format.

4.write_to_csv:
Writes user details to a CSV file named users.csv.
Takes a list of users (dictionaries) and writes each user's details as a new row in the file.

5.write_repositories_to_csv:
Writes repository information to a CSV file named repositories.csv.
Takes a list of repositories (dictionaries) and writes each repository’s details as a new row in the file.

6.get_user_repositories:
Takes a GitHub username and fetches their repositories (up to 500 repositories).
Retrieves repositories in batches of 100 and stops if there are fewer than 100 repositories in a response or the limit of 500 is reached.
Returns a list of repositories.


7.main():
Calls get_users_from_bangalore() to get a list of users.
Retrieves detailed information for each user by calling get_user_details() and stores this in user_details.
Writes the user details to users.csv via write_to_csv.
For each user, calls get_user_repositories() to retrieve their repositories and accumulates them in all_repositories.
Each repository is appended to all_repositories with additional details: login, full_name, created_at, stargazers_count, watchers_count, and language.
Writes all repository details to repositories.csv using write_repositories_to_csv.

8.Execution:
The script is executed by running main() if it is the main module (__name__ == "__main__")


**II.The most interesting and surprising fact you found after analyzing the the data:**
In user.csv : A unique and surprising insight is that one user has an exceptionally high number of public repositories, 1,563, which stands out as it’s nearly five times higher than most others in the dataset.

in Repositories.csv : Surprisingly, repositories written in Pascal have the highest average star count, with an average of 92 stars per repository, despite Pascal being a relatively rare language. The most common languages in these repositories are JavaScript, Python , and HTML . JavaScript’s dominance isn’t unexpected, but the high number of HTML repositories stands out, indicating many projects involve web-related or documentation work.


**III.An actionable recommendation for developers based on your analysis:**
Developers should consider specializing or contributing to niche programming languages or frameworks, like Pascal or Rust, which have been found to have higher average stars per repository.
