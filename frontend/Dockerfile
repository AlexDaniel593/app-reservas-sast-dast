FROM node:18-alpine AS builder

ARG NEXT_PUBLIC_API_URL
ARG NEXT_PUBLIC_BOOKING_URL
ARG NEXT_PUBLIC_USER_URL

WORKDIR /app

COPY package.json package-lock.json* ./
COPY . .

RUN echo "NEXT_PUBLIC_API_URL=$NEXT_PUBLIC_API_URL" > .env.production.local && \
    echo "NEXT_PUBLIC_BOOKING_URL=$NEXT_PUBLIC_BOOKING_URL" >> .env.production.local && \
    echo "NEXT_PUBLIC_USER_URL=$NEXT_PUBLIC_USER_URL" >> .env.production.local

RUN npm install
RUN npm run build

FROM node:18-alpine AS runner

WORKDIR /app

ENV NODE_ENV production

COPY --from=builder /app/public ./public
COPY --from=builder /app/.next ./.next
COPY --from=builder /app/package.json ./

RUN npm install --omit=dev

EXPOSE 3000

CMD ["npm", "start"]
