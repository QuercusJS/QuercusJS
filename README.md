
![Anurag's GitHub stats](https://github-readme-stats.vercel.app/api?username=QuercusJS&theme=dark&show_icons=true)
![Top Langs](https://github-readme-stats.vercel.app/api/top-langs/?username=QuercusJS&theme=tokyonight)
<img src="https://github.com/QuercusJS/QuercusJS/blob/main/Banner%20de%20LinkedIn%20Respuesta%20Global%20Ingeniera%20Amarillo%20y%20Negro.png">
<h1 align="center">Hi 👋, I'm Albert</h1>
<img src="https://github.com/QuercusJS/QuercusJS/blob/main/bongo-cat-explode-bongo-cat.gif" alt="" width="500" height="600">
<h3 align="center">A passionate Fullstack developer from Spain ;)</h3>

- 🔭 I’m currently working on **ticketchange**

- 🌱 I’m currently learning **React**

- 👯 I’m looking to collaborate on **None ;(**

- 🤝 I’m looking for help with **React devs**

- 💬 Ask me about **Laravel**

- 📫 How to reach me **indexado05@gmail.com**

- ⚡ Fun fact **i like Higuanas**

<h3 align="left">Connect with me:</h3>
<p align="left">
</p>

<h3 align="left">Languages and Tools:</h3>
<p align="left"> <a href="https://getbootstrap.com" target="_blank" rel="noreferrer"> <img src="https://raw.githubusercontent.com/devicons/devicon/master/icons/bootstrap/bootstrap-plain-wordmark.svg" alt="bootstrap" width="40" height="40"/> </a> <a href="https://www.w3schools.com/css/" target="_blank" rel="noreferrer"> <img src="https://raw.githubusercontent.com/devicons/devicon/master/icons/css3/css3-original-wordmark.svg" alt="css3" width="40" height="40"/> </a> <a href="https://www.figma.com/" target="_blank" rel="noreferrer"> <img src="https://www.vectorlogo.zone/logos/figma/figma-icon.svg" alt="figma" width="40" height="40"/> </a> <a href="https://git-scm.com/" target="_blank" rel="noreferrer"> <img src="https://www.vectorlogo.zone/logos/git-scm/git-scm-icon.svg" alt="git" width="40" height="40"/> </a> <a href="https://www.w3.org/html/" target="_blank" rel="noreferrer"> <img src="https://raw.githubusercontent.com/devicons/devicon/master/icons/html5/html5-original-wordmark.svg" alt="html5" width="40" height="40"/> </a> <a href="https://developer.mozilla.org/en-US/docs/Web/JavaScript" target="_blank" rel="noreferrer"> <img src="https://raw.githubusercontent.com/devicons/devicon/master/icons/javascript/javascript-original.svg" alt="javascript" width="40" height="40"/> </a> <a href="https://laravel.com/" target="_blank" rel="noreferrer"> <img src="https://raw.githubusercontent.com/devicons/devicon/master/icons/laravel/laravel-plain-wordmark.svg" alt="laravel" width="40" height="40"/> </a> <a href="https://www.mysql.com/" target="_blank" rel="noreferrer"> <img src="https://raw.githubusercontent.com/devicons/devicon/master/icons/mysql/mysql-original-wordmark.svg" alt="mysql" width="40" height="40"/> </a> <a href="https://www.php.net" target="_blank" rel="noreferrer"> <img src="https://raw.githubusercontent.com/devicons/devicon/master/icons/php/php-original.svg" alt="php" width="40" height="40"/> </a> <a href="https://reactjs.org/" target="_blank" rel="noreferrer"> <img src="https://raw.githubusercontent.com/devicons/devicon/master/icons/react/react-original-wordmark.svg" alt="react" width="40" height="40"/> </a> <a href="https://tailwindcss.com/" target="_blank" rel="noreferrer"> <img src="https://www.vectorlogo.zone/logos/tailwindcss/tailwindcss-icon.svg" alt="tailwind" width="40" height="40"/> </a> <a href="https://vuejs.org/" target="_blank" rel="noreferrer"> <img src="https://raw.githubusercontent.com/devicons/devicon/master/icons/vuejs/vuejs-original-wordmark.svg" alt="vuejs" width="40" height="40"/> </a> </p>
"""
   Copyright 2020-2022 Yufan You <https://github.com/ouuan>
   Licensed under the Apache License, Version 2.0 (the "License");
   you may not use this file except in compliance with the License.
   You may obtain a copy of the License at
       http://www.apache.org/licenses/LICENSE-2.0
   Unless required by applicable law or agreed to in writing, software
   distributed under the License is distributed on an "AS IS" BASIS,
   WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
   See the License for the specific language governing permissions and
   limitations under the License.
"""

import requests
import json
import sys
import re

if __name__ == "__main__":
    assert(len(sys.argv) == 4)
    handle = sys.argv[1]
    token = sys.argv[2]
    readmePath = sys.argv[3]

    headers = {
        "Authorization": f"token {token}"
    }

    followers = []
    cursor = None

    while True:
        query = f'''
query {{
    user(login: "{handle}") {{
        followers(first: 100{f', after: "{cursor}"' if cursor else ''}) {{
            pageInfo{{
                endCursor
                hasNextPage
            }}
            nodes {{
                login
                name
                databaseId
                following {{
                    totalCount
                }}
                repositories(first: 3, isFork: false, orderBy: {{
                    field: STARGAZERS,
                    direction: DESC
                }}) {{
                    totalCount
                    nodes {{
                        stargazerCount
                    }}
                }}
                followers {{
                    totalCount
                }}
            }}
        }}
    }}
}}
'''
        response = requests.post(f"https://api.github.com/graphql", json.dumps({ "query": query }), headers = headers)
        res = response.json()["data"]["user"]["followers"]
        for follower in res["nodes"]:
            following = follower["following"]["totalCount"]
            repoCount = follower["repositories"]["totalCount"]
            login = follower["login"]
            name = follower["name"]
            id = follower["databaseId"]
            followerNumber = follower["followers"]["totalCount"]
            thirdStars = follower["repositories"]["nodes"][2]["stargazerCount"] if repoCount >= 3 else 0
            if following > thirdStars * 50 + repoCount * 5 + followerNumber:
                print(f"Skipped: https://github.com/{login} with {followerNumber} followers and {following} following")
                continue
            followers.append((followerNumber, login, id, name if name else login))
            print(followers[-1])
        if not res["pageInfo"]["hasNextPage"]:
            break
        cursor = res["pageInfo"]["endCursor"]

    followers.sort(reverse = True)

    html = "<table>\n"

    for i in range(min(len(followers), 21)):
        login = followers[i][1]
        id = followers[i][2]
        name = followers[i][3]
        if i % 7 == 0:
            if i != 0:
                html += "  </tr>\n"
            html += "  <tr>\n"
        html += f'''    <td align="center">
      <a href="https://github.com/{login}">
        <img src="https://avatars2.githubusercontent.com/u/{id}" width="100px;" alt="{login}"/>
      </a>
      <br />
      <a href="https://github.com/{login}">{name}</a>
    </td>
'''

    html += "  </tr>\n</table>"

    with open(readmePath, "r") as readme:
        content = readme.read()

    newContent = re.sub(r"(?<=<!\-\-START_SECTION:top\-followers\-\->)[\s\S]*(?=<!\-\-END_SECTION:top\-followers\-\->)", f"\n{html}\n", content)

    with open(readmePath, "w") as readme:
        readme.write(newContent)
