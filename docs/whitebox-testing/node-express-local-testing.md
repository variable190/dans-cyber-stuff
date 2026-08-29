# Local Testing

## Setting up

Install dependencies
```bash
npm install # reads package.json to identify the required packages, installs them into the local node_modules folder
```
Run the project
```bash
npm run dev # runs the script named dev from the scripts section of package.json
```
Test an endpoint
```bash
curl -s -X POST -H "Content-Type: application/json" -d '{"email": "dan@cyber.com"}' http://localhost:5000/api/auth/authenticate // confirm server running correctly
```

## Check dependencies for vulnerabilities

```bash
npm audit
```
Review the output for:
- Severity (critical/high first)
- Whether a fix is available (`npm audit fix`)
- Whether the vulnerable package is actually reachable in the app (if not reachable it may not be exploitable)

Also useful:

```bash
npm outdated          # packages behind latest
cat package.json      # show direct dependencies (usually higher priority)
```

Note any packages that handle auth, cryptography, parsing, or user input as priority

## Debugging in VSCode/Codium

- Run and Debug tab
- Run icon next to "Launch Program"
- Click to left of the desired line to add a break point
- Trigger the breakpoint by sending the requried request
- Observe the variables and call stack at that point in the prrogram
- Use the run and debug toolbar to control the flow of the program (left to right):
    - **Continue** Resumes execution until the next breakpoint, exception, or the end of the request
    - **Step Over** Runs the current line and stops on the next line in the *same* function. Does not enter called functions
    - **Step Into** Enters the function called on the current line and stops on its first line
    - **Step Out** Finishes the current function and stops on the line after the call that entered it
    - **Restart** Stops the current debug session and starts it again from the beginning with the same launch config
    - **Stop** Ends the debug session immediately
    - **Dropdown Arrow - Disconnect** Detaches the debugger and leaves the process running