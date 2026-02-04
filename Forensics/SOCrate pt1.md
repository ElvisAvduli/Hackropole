# SOCrate 1/6 

**Challenge name:** SOCrate 1/6
**Description:**

> In June 2023, a vital operator was the victim of an attack that compromises its entire information system.
> You have received the Linux and Windows logs and you have to answer questions from the investigators.
> On the machine *webserver*, under what path does the web application turn?

**Goal:** Find the filesystem path where the web application is hosted.

---

## Step 1 — Extract the Evidence

We are provided with an archive containing the collected logs and artifacts. First, extract it:

```bash
tar -xvf socrate.tar.xz
```

This produces a directory tree containing various logs and files from the compromised systems.

---

## Step 2 — Search for Web Application Paths

The question asks specifically for the path of the web application on the **webserver** machine. A common Linux web root is under `/var/www`, so a good first reflex is to search for references to that path in the extracted data.

We can recursively grep for `/var/www/app`:

```bash
grep -rn "/var/www/app" .
```

This command:

* `-r`: searches recursively
* `-n`: shows line numbers
* `.`: searches in the current extracted directory

---

## Step 3 — Analyze the Results

The search returns hits pointing to the following path:

```
/var/www/app/banque_paouf/
```

This indicates that the web application is deployed in a subdirectory named `banque_paouf` under `/var/www/app`.

---

## Final Answer

**Flag:**

```
FCSC{/var/www/app/banque_paouf/}
```
