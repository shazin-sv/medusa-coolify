FROM node:23 as builder

WORKDIR /app/medusa

COPY . . 

RUN rm -rf node_modules

RUN apt-get update && apt-get install -y python3 python3-pip python-is-python3

RUN yarn install --loglevel=error

# This builds the server and the admin panel files
RUN yarn run build


FROM node:23

# Arguments for environment variables
ARG COOKIE_SECRET
ARG JWT_SECRET
ARG STORE_CORS
ARG ADMIN_CORS
ARG AUTH_CORS
ARG DISABLE_ADMIN
ARG WORKER_MODE
ARG PORT
ARG DATABASE_URL
ARG REDIS_URL
ARG BACKEND_URL

WORKDIR /app/medusa

# 1. FIXED: Uncommented and corrected these so the server has its "brain"
# These files are required for the admin panel to be served
COPY --from=builder /app/medusa/package.json .
COPY --from=builder /app/medusa/medusa-config.ts .
COPY --from=builder /app/medusa/tsconfig.json .
COPY --from=builder /app/medusa/yarn.lock .

RUN apt-get update && apt-get install -y python3 python3-pip python-is-python3

# 2. Copy the built files from the builder stage
COPY --from=builder /app/medusa/.medusa ./.medusa

WORKDIR /app/medusa/.medusa/server

# Install production dependencies
RUN yarn install --production

# 3. FIXED MIGRATIONS: 
# Instead of running migrations during 'build' (which fails due to DB connection),
# we run them at 'runtime' inside the CMD.
CMD ["sh", "-c", "npx medusa db:migrate && yarn run start"]