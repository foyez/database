PROJECT_NAME=databases-practice

up:
	docker compose up -d

down:
	docker compose down

restart:
	docker compose down
	docker compose up -d

logs:
	docker compose logs -f

ps:
	docker compose ps

clean:
	docker compose down -v
	docker system prune -f

# Database shells
psql:
	docker exec -it dev_postgres psql -U devuser -d devdb

mongo:
	docker exec -it dev_mongodb mongosh

redis:
	docker exec -it dev_redis redis-cli

.PHONY: up down restart logs ps clean psql mongo redis
