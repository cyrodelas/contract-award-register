# Contract Award Register - Smartsheet + Vercel

This repository is ready to deploy from GitHub to Vercel.

## Smartsheet sources

| Dataset | Sheet ID |
|---|---:|
| Project Type | 4544932179300228 |
| Project | 640707923496836 |
| Contract Packages | 1777432421158788 |
| Sub Packages | 3541014712373124 |
| Applicability | 4626949948526468 |
| Package Hierarchy | 2112606300229508 |

## Repository structure

```text
/
├── index.html
├── api/
│   └── smartsheet.js
├── package.json
├── vercel.json
├── .env.example
├── .gitignore
└── README.md
```

`index.html` is the Contract Award application. It calls `/api/smartsheet?sheetId=...`.

`api/smartsheet.js` is the Vercel serverless function. It holds no API token in source code. It reads the token from the Vercel environment and only allows the six configured sheet IDs.

## 1. Create a GitHub repository

1. Create a new GitHub repository.
2. Upload all files and folders from this project to the repository root.
3. Make sure the `api` folder is preserved.
4. Do not commit a real `.env` file or your Smartsheet API token.

## 2. Import the repository into Vercel

1. Sign in to Vercel.
2. Choose **Add New > Project**.
3. Import the GitHub repository.
4. Leave the framework preset as **Other** if Vercel does not identify a framework.
5. The project root should remain the repository root.

## 3. Add the Smartsheet API token

In Vercel:

**Project > Settings > Environment Variables**

Add:

```text
Name: SMARTSHEET_ACCESS_TOKEN
Value: <your Smartsheet API access token>
```

Add it to the environments you need, normally Production, Preview, and Development.

Redeploy after adding or changing the token.

## 4. Deploy

Click **Deploy**. Vercel will host `index.html` and deploy `api/smartsheet.js` as a serverless API endpoint.

Every future push to the connected GitHub branch can trigger a new Vercel deployment.

## Expected Smartsheet columns

### Project Type
- Type Code
- Type Name

### Project
- Project Code
- Project Name

### Contract Packages
- CP Row No.
- Contract Package Type
- CP Code
- CP Name

### Sub Packages
- CP Row No.
- Parent CP Type
- Parent CP Code
- Parent CP Name
- Subpackage Sequence
- Subpackage Code
- Subpackage Name

### Applicability
- CP Row No.
- Contract Package Type
- CP Code
- CP Name
- Applicable Project Type Code

### Package Hierarchy
The current front end retrieves this sheet and keeps it in `PACKAGE_HIERARCHY_MASTER`. Its existing columns can remain aligned with the extracted Package Hierarchy workbook.

## Security

Never place `SMARTSHEET_ACCESS_TOKEN` in `index.html`, JavaScript committed to GitHub, or any public repository file. The browser talks only to the same-origin Vercel API route; the API function attaches the bearer token server-side.

The API route also rejects sheet IDs that are not in the six-sheet allowlist.

## Current persistence behavior

Master data is read from Smartsheet. Contract Award register/draft/history records are still stored in browser `localStorage`, matching the existing prototype behavior.

If this is moved to multi-user production use, the next architectural step should be to persist Contract Award transactions in a shared database or a dedicated Smartsheet transaction sheet rather than browser storage.
