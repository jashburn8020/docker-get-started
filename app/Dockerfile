# syntax=docker/dockerfile:1
# FROM: initialize a new build stage and set the Base Image for subsequent instructions
FROM node:12-alpine

# RUN: Execute any commands in a new layer on top of the current image and commit the results
# The resulting committed image will be used for the next step
# Add packages to Alpine
RUN apk add --no-cache python2 g++ make

# WORKDIR: set the working directory for any RUN, CMD, ENTRYPOINT, COPY and ADD instructions that follow it
WORKDIR /app

# COPY: copy new files or directories from <src> and add them to the filesystem of the container at the path <dest>
COPY . .
RUN yarn install --production

# CMD: provide defaults for an executing container
# The default command to run when starting a container from this image
CMD ["node", "src/index.js"]

# EXPOSE: inform Docker that the container listens on the specified network ports at runtime
EXPOSE 3000
