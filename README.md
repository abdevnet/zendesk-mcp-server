# Zendesk MCP Server

![ci](https://github.com/reminia/zendesk-mcp-server/actions/workflows/ci.yml/badge.svg)
[![License](https://img.shields.io/badge/License-Apache_2.0-blue.svg)](https://opensource.org/licenses/Apache-2.0)

Forked from [reminia/zendesk-mcp-server](https://github.com/reminia/zendesk-mcp-server) with additional Help Center article tools.

A Model Context Protocol server for Zendesk.

This server provides a comprehensive integration with Zendesk. It offers:

- Tools for searching, retrieving, and managing Zendesk tickets and comments
- Specialized prompts for ticket analysis and response drafting
- Full access to the Zendesk Help Center articles as knowledge base

![demo](https://res.cloudinary.com/leecy-me/image/upload/v1736410626/open/zendesk_yunczu.gif)

## Setup

- build: `uv venv && uv pip install -e .` or `uv build` in short.
- setup zendesk credentials in `.env` file, refer to [.env.example](.env.example).
- configure in Claude desktop:

```json
{
  "mcpServers": {
      "zendesk": {
          "command": "uv",
          "args": [
              "--directory",
              "/path/to/zendesk-mcp-server",
              "run",
              "zendesk"
          ]
      }
  }
}
```

### Docker

A pre-built image is published to GitHub Container Registry on every push to `main`:

```bash
docker pull ghcr.io/abdevnet/zendesk-mcp-server:latest
```

Copy `.env.example` to `.env` and fill in your Zendesk credentials, then run:

```bash
docker run --rm --env-file /path/to/.env ghcr.io/abdevnet/zendesk-mcp-server:latest
```

Add `-i` when wiring the container to MCP clients over STDIN/STDOUT (Claude Code uses this mode). For daemonized runs, add `-d --name zendesk-mcp`.

#### Building locally

If you prefer to build from source:

```bash
docker build -t zendesk-mcp-server .
docker run --rm --env-file /path/to/.env zendesk-mcp-server
```

The image installs dependencies from `requirements.lock`, drops privileges to a non-root user, and expects configuration exclusively via environment variables.

#### Claude MCP Integration

To use the Dockerized server from Claude Code/Desktop, add an entry to Claude Code's `settings.json` similar to:

```json
{
  "mcpServers": {
    "zendesk": {
      "command": "/usr/local/bin/docker",
      "args": [
        "run",
        "--rm",
        "-i",
        "--env-file",
        "/path/to/zendesk-mcp-server/.env",
        "ghcr.io/abdevnet/zendesk-mcp-server:latest"
      ]
    }
  }
}
```

Adjust the paths to match your environment. After saving the file, restart Claude for the new MCP server to be detected.

## Resources

- zendesk://knowledge-base, get access to the whole help center articles.

## Prompts

### analyze-ticket

Analyze a Zendesk ticket and provide a detailed analysis of the ticket.

### draft-ticket-response

Draft a response to a Zendesk ticket.

## Tools

### get_tickets

Fetch the latest tickets with pagination support

- Input:
  - `page` (integer, optional): Page number (defaults to 1)
  - `per_page` (integer, optional): Number of tickets per page, max 100 (defaults to 25)
  - `sort_by` (string, optional): Field to sort by - created_at, updated_at, priority, or status (defaults to created_at)
  - `sort_order` (string, optional): Sort order - asc or desc (defaults to desc)

- Output: Returns a list of tickets with essential fields including id, subject, status, priority, description, timestamps, and assignee information, along with pagination metadata

### get_ticket

Retrieve a Zendesk ticket by its ID

- Input:
  - `ticket_id` (integer): The ID of the ticket to retrieve

### get_ticket_comments

Retrieve all comments for a Zendesk ticket by its ID

- Input:
  - `ticket_id` (integer): The ID of the ticket to get comments for

### create_ticket_comment

Create a new comment on an existing Zendesk ticket

- Input:
  - `ticket_id` (integer): The ID of the ticket to comment on
  - `comment` (string): The comment text/content to add
  - `public` (boolean, optional): Whether the comment should be public (defaults to true)

### create_ticket

Create a new Zendesk ticket

- Input:
  - `subject` (string): Ticket subject
  - `description` (string): Ticket description
  - `requester_id` (integer, optional)
  - `assignee_id` (integer, optional)
  - `priority` (string, optional): one of `low`, `normal`, `high`, `urgent`
  - `type` (string, optional): one of `problem`, `incident`, `question`, `task`
  - `tags` (array[string], optional)
  - `custom_fields` (array[object], optional)

### update_ticket

Update fields on an existing Zendesk ticket (e.g., status, priority, assignee)

- Input:
  - `ticket_id` (integer): The ID of the ticket to update
  - `subject` (string, optional)
  - `status` (string, optional): one of `new`, `open`, `pending`, `on-hold`, `solved`, `closed`
  - `priority` (string, optional): one of `low`, `normal`, `high`, `urgent`
  - `type` (string, optional)
  - `assignee_id` (integer, optional)
  - `requester_id` (integer, optional)
  - `tags` (array[string], optional)
  - `custom_fields` (array[object], optional)
  - `due_at` (string, optional): ISO8601 datetime

### search_tickets

Search Zendesk tickets by keyword query using the Zendesk Search API

- Input:
  - `query` (string): Search query (supports [Zendesk search syntax](https://support.zendesk.com/hc/en-us/articles/4408886879258) e.g. `status:open assignee:me`, `subject:keyword`)
  - `per_page` (integer, optional): Results per page, max 100 (defaults to 25)
  - `page` (integer, optional): Page number (defaults to 1)
  - `sort_by` (string, optional): Sort by field - created_at, updated_at, priority, or status (defaults to updated_at)
  - `sort_order` (string, optional): Sort order - asc or desc (defaults to desc)
- Output: Returns matching ticket metadata (id, subject, status, priority, timestamps, requester/assignee IDs) with pagination info

### get_help_center_sections

List all Help Center sections

- Input: None
- Output: Returns a list of sections with id, name, description, and URL

### get_section_articles

Get all articles in a specific Help Center section (metadata only)

- Input:
  - `section_id` (integer): The ID of the Help Center section
- Output: Returns article metadata (id, title, updated_at, URL) without body content

### get_article

Get a single Help Center article by ID, including full body content

- Input:
  - `article_id` (integer): The ID of the article to retrieve
- Output: Returns full article including HTML body content

### search_articles

Search Help Center articles by keyword query (metadata only)

- Input:
  - `query` (string): Search query string
  - `per_page` (integer, optional): Results per page, max 100 (defaults to 25)
  - `page` (integer, optional): Page number (defaults to 1)
- Output: Returns matching article metadata with pagination info. Use `get_article` to fetch full content for a specific result.

### update_article

Update an existing Help Center article (title, body, draft status, etc.)

- Input:
  - `article_id` (integer, required): The ID of the article to update
  - `title` (string, optional): New title for the article
  - `body` (string, optional): New HTML body content
  - `label_names` (array of strings, optional): Labels for the article
- Output: Returns the updated article. Articles are saved as drafts.

### update_article_from_markdown

Update a Help Center article from a local markdown file. Converts markdown to HTML automatically using the `extra` and `codehilite` extensions (supports tables, fenced code blocks, footnotes, etc.).

- Input:
  - `article_id` (integer, required): The ID of the article to update
  - `file_path` (string, required): Path to the local `.md` file
  - `title` (string, optional): Override article title (keeps existing if omitted)
- Output: Returns the updated article. Articles are saved as drafts.
