# Hexo CLI Cheatsheet

This file provides a quick reference for common Hexo commands.

## Common Commands

### Initialize a new website
Initializes a new Hexo project in the specified folder. If no folder is provided, it uses the current directory.
```bash
hexo init [folder]
```

### Create a new post
Creates a new post with the specified title. The new file will be located in `source/_posts/`.
```bash
hexo new [layout] <title>
```
- `layout`: The layout to use. Defaults to `post` if not specified. Other options are `page` and `draft`.
- `title`: The title of the post.

Example:
```bash
hexo new post "My New Post"
```

### Generate static files
Generates the static files for your website. The generated files are placed in the `public` directory.
```bash
hexo generate
```
Options:
- `-d`, `--deploy`: Deploy after generation.
- `-w`, `--watch`: Watch for file changes.

### Publish a draft
Publishes a draft from the `source/_drafts` folder.
```bash
hexo publish [layout] <filename>
```

### Run a local server
Starts a local server to preview your website. By default, the site is available at `http://localhost:4000`.
```bash
hexo server
```
Options:
- `-p`, `--port`: Port to use. Default is 4000.
- `-s`, `--static`: Serve static files only.
- `-l`, `--log`: Enable logger.

### Deploy your website
Deploys your website to the configured target (e.g., Git, Heroku). The deployment configuration is in `_config.yml`.
```bash
hexo deploy
```
Options:
- `-g`, `--generate`: Generate before deploying.

### Render files
Renders files to the `public` directory without generating the entire site.
```bash
hexo render <file1> [file2] ...
```

### Migrate content
Migrates content from other blogging systems.
```bash
hexo migrate <type>
```

### Clean the cache and generated files
Removes the `db.json` cache file and the `public` directory.
```bash
hexo clean
```

### List routes
Lists all the routes that will be generated.
```bash
hexo list <type>
```
- `type`: Can be `page`, `post`, `route`, `tag`, `category`.

### Display version information
Displays the version of Hexo and its plugins.
```bash
hexo version
```
