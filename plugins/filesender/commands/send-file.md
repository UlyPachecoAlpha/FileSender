# Send File to Power Automate

Send a file from the local filesystem to a Power Automate flow via its HTTP trigger endpoint.

## Usage

```
/power-automate-file-sender:send-file <file_path> [action]
```

## Arguments

- **file_path** (required): The absolute or relative path to the file you want to send.
- **action** (optional): A string describing what the Power Automate flow should do with the file. Defaults to `"upload"`.

## What this command does

1. Reads the file at the given path.
2. Determines the MIME content type from the file extension.
3. Base64-encodes the file contents.
4. Sends an HTTP POST request to the configured Power Automate flow endpoint with this JSON body:

```json
{
  "action": "<action>",
  "title": "<filename without extension>",
  "fileName": "<full filename>",
  "fileContent": "<base64-encoded file data>",
  "contentType": "<MIME type>"
}
```

5. Reports the response status back to the user.

## Instructions for Claude

When the user invokes this command:

1. Confirm the file path exists. If it doesn't, ask the user to provide a valid path.
2. Read the file as binary using a shell command:
   ```bash
   base64 -w 0 "<file_path>"
   ```
3. Determine the MIME type using:
   ```bash
   file --mime-type -b "<file_path>"
   ```
4. Extract the filename and title (filename without extension) using `basename` and shell parameter expansion.
5. Construct the JSON payload matching the schema:
   - `action`: The user-provided action string, or `"upload"` by default.
   - `title`: The filename without its extension.
   - `fileName`: The full filename including extension.
   - `fileContent`: The base64-encoded string from step 2.
   - `contentType`: The MIME type from step 3.
6. Send the POST request using `curl`:
   ```bash
   curl -s -w "\n%{http_code}" -X POST \
     -H "Content-Type: application/json" \
     -d @payload.json \
     "<FLOW_URL>"
   ```
   Where `<FLOW_URL>` is the Power Automate HTTP trigger URL from the skill file.
7. Report the result:
   - On success (2xx): Tell the user the file was sent successfully and include any response body.
   - On failure: Show the HTTP status code and response body so the user can troubleshoot.

## Examples

```
/power-automate-file-sender:send-file /home/user/reports/quarterly.pdf
/power-automate-file-sender:send-file ./proposal.docx "review"
/power-automate-file-sender:send-file ~/data/export.csv "import"
```
