# RamJam

This repo holds the documentation about our JamStack: Why and How.

# Context

Needs:

- Deploy multiple "small" websites
- Allow websites to have some dynamic content
- Provide to non-technical users the ability to update the dynamic
  content
- Reduce to its minimum the maintenance cost of the system

# The stack

- Deploy a single headless CMS, ie. a website that offers a user-friendly
interface to CRUD the data and an API to programatically fetch them
- generate as many static websites as needed and use the API to get the dynamic data

As the headless CMS we go with [Directus](https://directus.io).

To generate the static website we go with [Gatsby](https://www.gatsbyjs.org).

# Deploy a Directus instance

Use dokcer-compose:

```yaml
version: "3"
services:
  mysql:
    image: mysql:5.7
    environment:
      MYSQL_DATABASE: "directus"
      MYSQL_USER: "directus"
      MYSQL_PASSWORD: "directus"
      MYSQL_ROOT_PASSWORD: "directus"
    ports:
      - "3306:3306"

  directus:
    image: directus/directus:v8-apache
    ports:
      - "8080:80"
    environment:
      DIRECTUS_APP_ENV: "production"
      DIRECTUS_AUTH_PUBLICKEY: "some random secret"
      DIRECTUS_AUTH_SECRETKEY: "another random secret"
      DIRECTUS_DATABASE_HOST: "mysql"
      DIRECTUS_DATABASE_PORT: "3306"
      DIRECTUS_DATABASE_NAME: "directus"
      DIRECTUS_DATABASE_USERNAME: "directus"
      DIRECTUS_DATABASE_PASSWORD: "directus"
    volumes:
      - ./data/config:/var/directus/config
      - ./data/uploads:/var/directus/public/uploads
    links:
      - mysql:mysql
```

# Generate a static website

```js
# pages/a-gatsby-page.js
import React, { useState, useEffect } from "react"
import Layout from "../components/layout"

export default ({ data }) => {

  const [title, setTitle] = useState(0)
  const [content, setContent] = useState(1)
  useEffect(() => {
    fetch(`http://0.0.0.0:8080/Directus/items/bullenetworkpages/1`)
      .then(response => response.json())
      .then(resultData => {
        setTitle(resultData.data.title)
        setContent(resultData.data.content)
      })
  }, [])

  return (
    <Layout>
      <h1>{title}</h1>
      <div dangerouslySetInnerHTML={{__html: content}} />
    </Layout>
  )
}
```

# Ubuntu installation

Install docker

```
# from https://docs.docker.com/install/linux/docker-ce/ubuntu/
sudo apt-get remove docker docker-engine docker.io containerd runc
sudo apt-get update
sudo apt-get install \
    apt-transport-https \
    ca-certificates \
    curl \
    gnupg-agent \
    software-properties-common
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
sudo apt-key fingerprint 0EBFCD88
> 9DC8 5822 9FC7 DD38 854A E2D8 8D81 803C 0EBF CD88
sudo add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
   $(lsb_release -cs) \
   stable"
sudo apt-get update
```

Install docker-compose

```
# change the version accordingly
sudo curl -L "https://github.com/docker/compose/releases/download/1.25.3/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
sudo ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose
```

Clone, set and run

```
git clone https://github.com/doxa-tech/bullenetwork-directus.git
cd bullenetwork-directus
cp directus.env.template directus.env
cp mysql.env.template mysql.env
> set the variables in those files
# Pull the latest images
sudo docker-compose pull
#  Run the stack
docker-compose up -d
#  Initialize the database and an admin user
docker-compose run directus install --email email@example.com --password d1r3ctu5
```
