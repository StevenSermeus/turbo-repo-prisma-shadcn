FROM node:20-alpine AS base
 
FROM base AS builder
RUN apk update
RUN apk add --no-cache libc6-compat
# Set working directory
WORKDIR /app
# Replace <your-major-version> with the major version installed in your repository. For example:
# RUN yarn global add turbo@^2
RUN yarn global add turbo
COPY . .
 
# Generate a partial monorepo with a pruned lockfile for a target workspace.
# Assuming "web" is the name entered in the project's package.json: { name: "web" }
RUN turbo prune pure-life --docker
 
# Add lockfile and package.json's of isolated subworkspace
FROM base AS installer
RUN apk update
RUN apk add --no-cache libc6-compat
WORKDIR /app
 
# First install the dependencies (as they change less often)
COPY --from=builder /app/out/json/ .
COPY --from=builder /app/packages/database/prisma/schema.prisma ./packages/database/prisma/schema.prisma
RUN yarn install
 
# Build the project
COPY --from=builder /app/out/full/ .

RUN SKIP_ENV_VALIDATION=1 yarn turbo run build --filter=pure-life...
 
FROM base AS runner
WORKDIR /app
RUN apk add --no-cache curl 
# Don't run production as root
RUN addgroup --system --gid 1001 nodejs
RUN adduser --system --uid 1001 nextjs
USER nextjs
 
# Automatically leverage output traces to reduce image size
# https://nextjs.org/docs/advanced-features/output-file-tracing
COPY --from=installer --chown=nextjs:nodejs /app/apps/pure-life/.next/standalone ./
COPY --from=installer --chown=nextjs:nodejs /app/apps/pure-life/.next/static ./apps/pure-life/.next/static
COPY --from=installer --chown=nextjs:nodejs /app/apps/pure-life/public ./apps/pure-life/public
 
CMD node apps/pure-life/server.js