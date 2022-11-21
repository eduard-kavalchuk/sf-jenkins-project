Jenkins pipeline aimed to do the following:
- Wait for a commit to a specified repository.
- Start an Nginx container.
- Get a HTTP response from a container.
- Fetch an index web page.
- Compare the web page's md5 hash with the original page in the repo.
- Delete a container.
In case of any errors send a message to Telegram and stop a pipeline.
