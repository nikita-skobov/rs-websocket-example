# rs-websocket-example
an example of a websocket server and hosting instructure



## Instructions

This server does not use SSL, but rather relies on something in front of it to handle SSL termination.
These instructions setup a cloudfront CDN in front, but can be anything as long as it properly forwards headers and can support websocket sticky sessions.


1. Deploy a VM, in this example I am using AWS EC2, but you can use anything as long as it can be pointed to by AWS CloudFront.
2. VM needs to allow TCP traffic on port 3000 (the server binary we will build will listen on 3000, but you can modify that by editing `src/main.rs`)
3. Build the server binary.
   - If the VM is using an arm processor: `cross build --target aarch64-unknown-linux-musl --release` (see instructions for using cross: https://github.com/cross-rs/cross)
   - If the VM is using an x86 processor: `cross build --target x86_64-unknown-linux-musl --release`
   - Alternatively if you don't wish to build with cross, you can ssh into the VM and build it on the VM with just `cargo build --release` (but of course you'd have to install the rust toolchain on the VM)
4. Upload the server binary. `scp -i ~/path/to/cert.pem ./target/aarch64-unknown-linux-musl/release/websockettest ec2-user@[ip]:/home/ec2-user/`
5. ssh into the VM, and run the binary `ssh -i ~/path/to/cert.pem ec2-user@[ip]` and then `./websockettest`
6. test it out by loading it directly in browser: open `[ip]:3000` in your browser and should see a chat window. this will work but will only use http.
7. to connect via https we need something in front of our VM that will handle terminating SSL connections for us, in this case cloudfront. Create a new distribution in the AWS Console
8. enter the hostname of the VM you deployed. if using ec2 it should look something like this: ![image](https://github.com/user-attachments/assets/a5d89a2b-35ef-4d4e-b640-cce6807ee979)
9. set these cache policies: we aren't using Cloudfront as a CDN so we don't care about caching in this instance, and we want to ensure that headers get properly forwarded (needed for the websocket handshake) ![image](https://github.com/user-attachments/assets/8a88b497-9f56-42e1-9ab6-6077b7b28177)
10. other settings can be left as-is, and then create distribution. wait for it to finish deploying and then you can access the server via the cloudfront distribution hostname, it looks something like `https://dnabcd37c123a.cloudfront.net`


The html page served is embedded inside `src/main.rs`, and contains some javascript code that will initialize the websocket connection. It optionally uses `wss://` if the page is served over https, or otherwise uses `ws://`
