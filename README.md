# RAG SQL Query App

A modern web application that uses AI to generate and execute SQL queries based on natural language input. Powered by **Nuxt.js**, **Tailwind CSS**, **LangChain**, **Gemini**, and **Supabase**, this app allows users to query `users` and `products` data stored in a PostgreSQL database with an intuitive, responsive interface.

## Features

- **Natural Language to SQL**: Enter queries like "Show all users" or "List products under $50," and the app generates corresponding SQL using Gemini’s AI model.
- **Modern UI**: A sleek, card-based interface with gradients, animations, and a responsive table for query results, styled with Tailwind CSS.
- **Supabase Backend**: Securely stores and queries data using a custom `execute_sql` function in PostgreSQL.
- **Debugging**: Extensive logging in the frontend and backend for easy troubleshooting.
- **Extensible**: Ready for enhancements like query history, SQL toggling, or advanced security.

## Tech Stack

- **Frontend**: Nuxt.js 3, Vue.js, Tailwind CSS
- **Backend**: Nuxt Server Routes, Supabase (PostgreSQL)
- **AI**: LangChain, Google Gemini (gemini-1.5-pro)
- **Environment**: Node.js, npm

## Prerequisites

Before setting up the app, ensure you have:

- **Node.js** (v16 or higher): [Download](https://nodejs.org/)
- **npm** (comes with Node.js)
- **Supabase Account**: Sign up at [app.supabase.com](https://app.supabase.com)
- **Google Gemini API Key**: Obtain from [Google AI Studio](https://aistudio.google.com/)

## Installation

1. **Clone the Repository**:
   ```bash
   git clone https://github.com/your-username/rag-sql-query-app.git
   cd rag-sql-query-app
   ```

2. **Install Dependencies**:
   ```bash
   npm install
   ```

3. **Set Up Environment Variables**:
   - Create a `.env` file in the project root:
     ```env
     SUPABASE_URL=https://your-project.supabase.co
     SUPABASE_KEY=your-anon-key
     GOOGLE_API_KEY=your-gemini-api-key
     ```
   - Replace placeholders with:
     - `SUPABASE_URL` and `SUPABASE_KEY` from your Supabase project’s **Settings > API**.
     - `GOOGLE_API_KEY` from Google AI Studio.

4. **Configure Supabase**:
   - Log in to [Supabase Dashboard](https://app.supabase.com).
   - Create tables for `users` and `products`:
     ```sql
     CREATE EXTENSION IF NOT EXISTS "uuid-ossp";

     CREATE TABLE users (
       id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
       name TEXT NOT NULL,
       email TEXT NOT NULL,
       created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP
     );

     CREATE TABLE products (
       id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
       name TEXT NOT NULL,
       price NUMERIC NOT NULL,
       created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP
     );
     ```
   - Insert sample data:
     ```sql
     INSERT INTO users (name, email) VALUES
       ('Alice Smith', 'alice@example.com'),
       ('Bob Jones', 'bob@example.com');

     INSERT INTO products (name, price) VALUES
       ('Laptop', 999.99),
       ('Headphones', 49.99);
     ```
   - Create the `execute_sql` function:
     ```sql
     DROP FUNCTION IF EXISTS execute_sql;
     CREATE OR REPLACE FUNCTION execute_sql(query TEXT)
     RETURNS JSON AS $$
     DECLARE
       result JSON;
       clean_query TEXT;
     BEGIN
       clean_query := TRIM(TRAILING ';' FROM TRIM(query));
       EXECUTE 'SELECT json_agg(t) FROM (' || clean_query || ') t' INTO result;
       RETURN result;
     EXCEPTION WHEN OTHERS THEN
       RAISE EXCEPTION 'Invalid query: %', SQLERRM;
     END;
     $$ LANGUAGE plpgsql;
     GRANT EXECUTE ON FUNCTION execute_sql TO public;
     ```

## Running the App

1. **Start the Development Server**:
   ```bash
   npm run dev
   ```
   - Open [http://localhost:3000](http://localhost:3000) in your browser.

2. **Build for Production**:
   ```bash
   npm run build
   npm run start
   ```

## Usage

1. **Enter a Query**:
   - In the input field, type a natural language query, e.g., "Show all users" or "List products under $50."
   - Press **Enter** or click **Submit Query**.

2. **View Results**:
   - The app generates SQL (e.g., `SELECT * FROM users;`) using Gemini.
   - Results appear in a modern table with columns like `id`, `name`, `price`, etc.
   - Errors (if any) display in a styled red alert.

3. **Debugging**:
   - Check **browser Console** (F12) for frontend logs (e.g., `API response: ...`).
   - Check **terminal** for backend logs (e.g., `SQL execution result: ...`).

## Project Structure

```
rag-sql-query-app/
├── assets/
│   └── css/tailwind.css        # Tailwind CSS configuration
├── pages/
│   └── index.vue              # Main UI with modern design
├── server/
│   └── api/
│       └── rag.post.js        # Backend route for query processing
├── .env                       # Environment variables (not tracked)
├── nuxt.config.js             # Nuxt configuration
├── package.json               # Dependencies and scripts
└── README.md                  # This file
```

## Known Issues

- **Semicolon Error in Supabase**:
  - The `execute_sql` function may fail with `syntax error at or near ";"` for queries like `SELECT * FROM products;`.
  - **Workaround**: Ensure the `execute_sql` function includes `TRIM(TRAILING ';' FROM TRIM(query))`. Re-run:
    ```sql
    DROP FUNCTION IF EXISTS execute_sql;
    CREATE OR REPLACE FUNCTION execute_sql(query TEXT)
    RETURNS JSON AS $$
    DECLARE
      result JSON;
      clean_query TEXT;
    BEGIN
      clean_query := TRIM(TRAILING ';' FROM TRIM(query));
      EXECUTE 'SELECT json_agg(t) FROM (' || clean_query || ') t' INTO result;
      RETURN result;
    EXCEPTION WHEN OTHERS THEN
      RAISE EXCEPTION 'Invalid query: %', SQLERRM;
    END;
    $$ LANGUAGE plpgsql;
    GRANT EXECUTE ON FUNCTION execute_sql TO public;
    ```
  - Verify with:
    ```sql
    SELECT execute_sql('SELECT * FROM products;');
    ```
  - Check backend cleaning in `server/api/rag.post.js`:
    ```javascript
    const cleanSqlQuery = sqlQuery.trimEnd().replace(/;*\s*$/, '');
    ```

- **Empty Results**:
  - If tables are empty, insert data (see **Configure Supabase**).

## Troubleshooting

- **Supabase Errors**:
  - Verify `SUPABASE_URL` and `SUPABASE_KEY` in `.env`.
  - Test connection:
    ```javascript
    const { data, error } = await supabase.from('users').select('*');
    console.log({ data, error });
    ```

- **Gemini Errors**:
  - Ensure `GOOGLE_API_KEY` is valid.
  - Check logs: `Extracted SQL query: ...`.

- **UI Issues**:
  - Confirm Tailwind is set up:
    ```javascript
    // nuxt.config.js
    export default defineNuxtConfig({
      css: ['~/assets/css/tailwind.css'],
      modules: ['@nuxtjs/tailwindcss'],
    });
    ```

- **Logs**:
  - Frontend: Browser Console (F12).
  - Backend: Terminal running `npm run dev`.

## Future Enhancements

- **Query History**: Store queries in a `query_history` table:
  ```sql
  CREATE TABLE query_history (
    id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    query TEXT,
    sql_query TEXT,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP
  );
  ```
  ```javascript
  await supabase.from('query_history').insert({ query, sql_query: cleanSqlQuery });
  ```

- **Toggle SQL Display**: Show/hide generated SQL:
  ```vue
  <button @click="showSql = !showSql">Toggle SQL</button>
  <div v-if="showSql && sqlQuery">...</div>
  ```

- **Query Suggestions**:
  ```vue
  <input v-model="query" list="suggestions" />
  <datalist id="suggestions">
    <option value="Show all users" />
    <option value="List products under $50" />
  </datalist>
  ```

- **Security**: Restrict `execute_sql` to `SELECT` queries:
  ```sql
  IF clean_query ILIKE 'SELECT%' THEN ... ELSE RAISE EXCEPTION 'Only SELECT queries allowed';
  ```

## Contributing

1. Fork the repository.
2. Create a branch: `git checkout -b feature/your-feature`.
3. Commit changes: `git commit -m "Add your feature"`.
4. Push: `git push origin feature/your-feature`.
5. Open a pull request.

## License

[MIT License](LICENSE) (or specify your preferred license).

## Contact

For issues or suggestions, open a GitHub issue or contact [your-email@example.com].

---

### Notes on the README
- **Repository**: Assumed `rag-sql-query-app`. Replace with your actual repo URL.
- **License**: Suggested MIT; add a `LICENSE` file or specify another.
- **Known Issues**: Included the semicolon error with steps to verify `execute_sql`, referencing your latest logs (`syntax error at or near ";"`).
- **Structure**: Matches your app (`pages/index.vue`, `server/api/rag.post.js`).
- **Supabase Setup**: Detailed table creation and `execute_sql` to help new users.
- **Troubleshooting**: Addresses common errors (e.g., Supabase connection, Gemini).
- **Enhancements**: Suggests features you’ve shown interest in (e.g., query history).

### Addressing the Semicolon Error
Since your latest error persists, the README includes a workaround for `execute_sql`. To resolve it immediately:

1. **Re-run `execute_sql`**:
   - In Supabase SQL Editor:
     ```sql
     DROP FUNCTION IF EXISTS execute_sql;
     CREATE OR REPLACE FUNCTION execute_sql(query TEXT)
     RETURNS JSON AS $$
     DECLARE
       result JSON;
       clean_query TEXT;
     BEGIN
       clean_query := TRIM(TRAILING ';' FROM TRIM(query));
       EXECUTE 'SELECT json_agg(t) FROM (' || clean_query || ') t' INTO result;
       RETURN result;
     EXCEPTION WHEN OTHERS THEN
       RAISE EXCEPTION 'Invalid query: %', SQLERRM;
     END;
     $$ LANGUAGE plpgsql;
     GRANT EXECUTE ON FUNCTION execute_sql TO public;
     ```
   - Test:
     ```sql
     SELECT execute_sql('SELECT * FROM products;');
     ```

2. **Fix Backend Cleaning**:
   - Update `server/api/rag.post.js`:
     ```javascript
     const cleanSqlQuery = sqlQuery.trimEnd().replace(/;*\s*$/, '');
     console.log('Cleaned SQL query (semicolon and whitespace removed):', cleanSqlQuery);
     const { data, error } = await supabase.rpc('execute_sql', { query: cleanSqlQuery });
     return { sqlQuery: cleanSqlQuery, results: data || [] };
     ```

3. **Verify**:
   - Run `npm run dev`.
   - Submit “Show all users”.
   - Check logs for:
     ```
     Cleaned SQL query (semicolon and whitespace removed): SELECT * FROM users
     SQL execution result: { data: [...], error: null }
     ```

### Customizing the README
- **Repository**: Replace `your-username/rag-sql-query-app` with your GitHub repo.
- **Contact**: Add your email or remove if not needed.
- **Enhancements**: I included query history and SQL toggle based on our discussions. Add others (e.g., dark mode) if desired.
- **Logo**: Add a logo at the top:
  ```markdown
  ![RAG SQL Query App](path/to/logo.png)
  ```

### Next Steps
- **Add README**:
  - Create `README.md` in your project root.
  - Copy-paste the above content.
  - Update placeholders (repo, email, license).

- **Fix Semicolon**:
  - Apply the `execute_sql` and backend fixes.
  - Test with the modern UI from your last message.

- **Enhance**:
  - Implement query history or suggestions from the README.
  - Uncomment `sqlQuery` in `pages/index.vue` with a toggle.
