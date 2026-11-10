# Forno — Pizzaria

Aplicação separada em frontend React + TypeScript, API Node.js + Express + TypeScript e banco MySQL 8.4. O Docker Compose inicia os três serviços juntos.

## Requisitos

- Docker Desktop ou Docker Engine com o plugin Docker Compose.
- Git (para clonar o repositório).

Não é necessário instalar Node.js ou MySQL na máquina para executar pelo Docker.

## Executar o projeto

1. Clone o repositório e entre na pasta:

   ```bash
   git clone <URL_DO_REPOSITORIO>
   cd pizzaria
   ```

2. Crie seu arquivo de ambiente:

   ```bash
   cp .env.example .env
   ```

   Em seguida, altere as senhas do arquivo `.env`.

3. Suba frontend, backend e MySQL:

   ```bash
   docker compose up --build
   ```

4. Acesse:

   - Frontend: http://localhost:5173
   - API (health check): http://localhost:3000
   - Health check: http://localhost:3000/health
   - Produtos: http://localhost:3000/api/products
   - Categorias: http://localhost:3000/api/categories

Para executar em segundo plano: `docker compose up --build -d`.
Para parar: `docker compose down`.
Para acompanhar logs: `docker compose logs -f`.

O MySQL cria as tabelas e os produtos iniciais na primeira inicialização. Os dados ficam no volume Docker `mysql_data`, portanto continuam disponíveis depois de parar e iniciar os containers. Para apagar também os dados: `docker compose down -v` (operação destrutiva).

## Estrutura

```text
pizzaria/
├── frontend/                 # React + Vite + TypeScript
│   └── src/
│       ├── components/
│       ├── context/
│       ├── pages/
│       └── services/api.ts
├── backend/                  # API Node + Express + TypeScript
│   ├── database/init.sql
│   └── src/
│       ├── config/
│       ├── controllers/
│       ├── middlewares/
│       ├── repositories/
│       ├── routes/
│       └── services/
├── docker-compose.yml
├── .env.example
└── package.json
```

## Endpoints implementados

| Método | Rota | Descrição |
|---|---|---|
| GET | `/health` | Verifica se a API está online |
| GET | `/api/products` | Lista produtos ativos com suas categorias |
| GET | `/api/categories` | Lista categorias ativas |
| POST | `/api/orders` | Valida e registra pedido e itens em uma transação MySQL |

### Exemplo de criação de pedido

`POST http://localhost:3000/api/orders`

```json
{
  "customerName": "Maria Silva",
  "customerPhone": "11999999999",
  "orderType": "ENTREGA",
  "deliveryAddress": "Rua das Flores, 100",
  "paymentMethod": "PIX",
  "items": [
    { "productId": 1, "quantity": 2 },
    { "productId": 3, "quantity": 1 }
  ]
}
```

O backend consulta os preços no banco e calcula subtotal, taxa de entrega e total. Não confia nos valores enviados pelo navegador. A taxa de entrega inicial está definida em R$ 8,00 no serviço de pedidos.

## Desenvolvimento sem Docker para frontend/backend

É possível instalar dependências separadamente (`npm install` dentro de `frontend/` e `backend/`) e executar `npm run dev` em cada pasta. Nesse modo, o MySQL precisa estar disponível e as variáveis do backend devem apontar para ele. O modo recomendado para iniciar tudo junto é o Docker Compose.

## Executar pelo GitHub Codespaces

1. Envie este projeto para um repositório GitHub.
2. Abra o repositório e selecione **Code → Codespaces → Create codespace**.
3. No terminal do Codespace, execute `docker compose up --build`.
4. Abra a porta `5173` na aba **Ports**. O Vite encaminha `/api` internamente para o backend, então o navegador não precisa conhecer o endereço interno do container.

## GitHub: o que ele executa

O GitHub armazena o código e executa a verificação automática de build pelo GitHub Actions. O GitHub, por si só, não mantém uma API Node e um banco MySQL disponíveis para usuários finais. Para publicar o sistema na internet, frontend, backend e MySQL precisam ser hospedados em serviços de execução/banco, configurando as variáveis de ambiente e a URL pública da API.


## Histórico de pedidos

- `GET /api/orders`: retorna o histórico de pedidos com itens, valores, forma de recebimento e status.
- `POST /api/orders`: registra novos pedidos (mantido).
- Na primeira inicialização do backend, são criados três pedidos demonstrativos. A rotina usa a tabela `app_seeds` para não duplicar os exemplos em reinicializações futuras, inclusive quando o volume MySQL já existe.
- A página **Meus pedidos** está disponível no menu principal em `/pedidos`.
# pizzaria-fatec
