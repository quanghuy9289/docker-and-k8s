# Specify a base image
FROM node:alpine

# from node version 15, WORKDIR need to specified
# if not it will through error: "npm ERR! Tracker "idealTree" already exists"
WORKDIR /usr/app

# Copy package.json
COPY ./package.json ./

# Install some dependencies
RUN npm install

# Copy all source to working directory above
# Do it after installing dependencies to avoid unnessessary re-install step if we just update code
COPY ./ ./

# Default command
CMD ["npm", "start"]