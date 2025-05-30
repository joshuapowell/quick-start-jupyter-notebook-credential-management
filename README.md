# Securely Handling Credentials in Local Code Notebooks

## Abstract

This document addresses the significant security risks associated with embedding credentials directly into Jupyter Notebooks, a common practice in data science and of importance to anyone interacting with secured REST APIs using Jupyter Notebooks. Hard coding credentials increases the likelihood of exposure through various channels, including unencrypted transmission, accidental commits to version control, and unauthorized user access, potentially violating corporate security policies. The plaintext nature of `.ipynb` files further exacerbates this issue, as sensitive information, including HTTP response data, can be easily extracted from the underlying JSON structure.

To mitigate these risks, the document explores several secure credential management options for Local Jupyter Notebooks. These range from using interactive prompts and environment variables to leveraging configuration files and cloud-based secrets managers such as AWS Secrets Manager, Google Cloud Secret Manager, and HashiCorp Vault. The document emphasizes the importance of adhering to best practices like avoiding hard coding, excluding credentials files from version control, employing encryption, and following the principle of least privilege to ensure the confidentiality and integrity of sensitive information.

**Keywords:** Jupyter Notebooks, Secret Management, Data Science, API, Integration, Enterprise

## 1. What are the risks of embedding credentials in a code notebook?
The risks of embedding credentials in a code notebook are the same as with any software development project and the practice should be avoided. When credentials are embedded or hard coded in code, it dramatically increases the chance of security breaches, including transmitting those credentials unencrypted, accidentally adding them to version control, granting unauthorized user access, and violating corporate policies.

## 2. How can credentials in a code notebook be compromised?
The consequences of exposing the credentials to a malicious third-party are clear to most developers, however, the manner in which those credentials are exposed through a code notebook are not always readily apparent.

The notebook, the `.ipynb` file, is displayed as a sort of "user interface" in most IDEs like Visual Studio Code, IntelliJ, JupyterLab, Google Collab, or other notebook tools. The `.ipynb` file is really just a JSON file with a custom extension that Jupyter understands. Therefore, anything you type in plaintext and save to the notebook will be saved to the notebook in plaintext.

As a result, the credentials, and to that end any data within the JSON file, have the ability to be exposed by reading the underlying plaintext JSON file.

**Note:** While this document is focused on credentials and not sensitive data, it's important to remember that the same rules apply with HTTP response payloads that are rendered in the notebook. Any data that is part of that payload, sensitive or not, has the potential to be saved to the file. This is important to remember when you're checking your notebook into version control and doubly important when you're considering how you share your notebook.

## 3. What options exist for handling credentials securely within notebooks?

### 3.1 Use Interactive Prompts
Using the `getpass` library to prompt for credentials interactively at runtime, circumvents the need to store credentials anywhere in code. While the name of the primary method is `getpass` it can be used to collect any sensitive string of information such as a username, password, or API Key.

```python
import getpass

username = getpass.getpass("Enter Username: ")
password = getpass.getpass("Enter Password: ")

api_key = getpass.getpass("Enter API Key: ")
```

See [example notebook with executable code here](./examples/credential-management-interactive-prompt.ipynb).

### 3.2 Use Environment Variables
Using environment variables is a fast way to access credentials from within a notebook without storing the credentials inside of the notebook code. Using this method prevents you from hard coding credentials directly in your code and avoids an accidental exposure to your repository via version control; the environment variables are most likely stored in plain text on your local development environment.

The `getenv` method can be used to retrieve any information stored as an environment variable in your operating system accessible to your Python environment.

```python
import os

username = os.getenv("USERNAME")
password = os.getenv("PASSWORD")

api_key = os.getenv("API_KEY")
```

See [example notebook with executable code here](./examples/credential-management-environment-variables.ipynb).

### 3.3 Configuration Files
Using a configuration file through a `.env`, `.json`, or `.yml` file is convenient and provides another layer of protection over simply storing your credentials in your operating system environment. It is important to make sure that you add this file to your `.gitignore` to prevent accidentally uploading it to version control.

```python
import os
from dotenv import load_dotenv
load_dotenv()

username = os.getenv("USERNAME")
password = os.getenv("PASSWORD")

api_key = os.getenv("API_KEY")
```

See [example notebook with executable code here](./examples/credential-management-configuration-file.ipynb).

Note: The example above requires a `.env` file to exist in the repository and requires that you install the `dotenv` Python package.

### 3.4 Secrets Managers
Using a cloud-based secret manager provides a few added benefits. First, the credentials are stored in an encrypted manner off-machine. Second, it reduces the risk of those secrets being exposed to version control by mistake. Finally, it moves the management of those secrets from you and your machine to your organization and it's cloud security policies.

#### 3.4.1 Examples include
- [Amazon Web Services (AWS), Secrets Manager](https://aws.amazon.com/secrets-manager/)
- [Google Cloud Platform (GCP), Secrets Manager](https://cloud.google.com/security/products/secret-manager)
- [HashiCorp Vault](https://www.hashicorp.com/en/products/vault)
- [IBM Cloud Secret Manager](https://www.ibm.com/products/secrets-manager)
- [Microsoft Azure](https://azure.microsoft.com/en-us/products/key-vault/)

Each cloud provider has its own Python SDK and methods of accessing secrets within your notebook. As an example the Google Cloud Platform implementation might look like this [2]:

```python
def get_secret(project_id: str, secret_id: str) -> secretmanager.GetSecretRequest:
    """
    Get information about the given secret. This only returns metadata about
    the secret container, not any secret material.
    """

    # Import the Secret Manager client library.
    from google.cloud import secretmanager

    # Create the Secret Manager client.
    client = secretmanager.SecretManagerServiceClient()

    # Build the resource name of the secret.
    name = client.secret_path(project_id, secret_id)

    # Get the secret.
    response = client.get_secret(request={"name": name})

    # Get the replication policy.
    if "automatic" in response.replication:
        replication = "AUTOMATIC"
    elif "user_managed" in response.replication:
        replication = "MANAGED"
    else:
        raise Exception(f"Unknown replication {response.replication}")

    # Print data about the secret.
    print(f"Got secret {response.name} with replication policy {replication}")
```

### 3.5 Other Third-Party Options

- **Encrypted Files** using [`cryptography`](https://pypi.org/project/cryptography/)
- **Keyrings** using [`keyring`](https://pypi.org/project/keyring/)
- **Cloud Based Notebooks[3]** using services like [Google Collab](https://colab.research.google.com/) or [DeepNote](https://deepnote.com/).

### 3.6 Important Considerations
Using one of the recommended methods above, you can greatly reduce the possibility of leaking your credentials. 

**Warning**  All content entered or displayed via a "Run" command in a Jupyter Notebook is added to the `.ipynb` file. Therefore, if sensitive information is being requested from an external resource and you check-in that information to a repository it is made public to anyone with access to that repository.

## 4. What is required for me to test these methods?
Readers wishing to follow along with the concepts outlined in this document may wish to prepare their local development environment with an up to date version of [Microsoft Visual Studio Code](https://code.visualstudio.com/) and [Jupyter Extension for Visual Studio Code](https://marketplace.visualstudio.com/items?itemName=ms-toolsai.jupyter)or a comparable IDE with Jupyter extension in addition to the latest available version of [Python version 3](https://www.python.org/) running under [`pyenv`](https://github.com/pyenv/pyenv). 

Note: It is not recommended to use the Python version that ships with your operating system.

## 5. Conclusion
The hope is that by following one of the above methods, a combination of these methods, or identifying new methods that your credentials will remain secret.

- **Avoid Hard coding:** Never hard code credentials directly in your Jupyter Notebooks by using one of the above methods.
- **Version Control:** Exclude credentials files from version control using `.gitignore`.
- **Encryption:** Use encryption whenever possible to protect your credentials.
- **Principle of Least Privilege:** Grant only the necessary permissions to access your credentials.
- **Secure Notebook Server:** When Jupyter Notebooks are served over a public network ensure that the Jupyter Notebook Server is password protected and uses a valid SSL/TLS certificate.
- **Follow Security Policies:** Follow all of your organizations stated and recommended security policies.

Applying one of the above methods with these best practices will reduce the risk of credential exposure and the overall risk of exposing confidential data when working with code notebooks.

## 6. Further Reading
1. Souly, A. (2025, January 23). Keeping credentials safe in Jupyter Notebooks. Towards Data Science. https://towardsdatascience.com/keeping-credentials-safe-in-jupyter-notebooks-fbd215a8e311/
2. Get secret. (Accessed May 30, 2025). Secret Manager Documentation  |  Google Cloud. https://cloud.google.com/secret-manager/docs/samples/secretmanager-get-secret?hl=en#secretmanager_get_secret-python
3. Tate, A. (2023, October 11). Working Securely with Jupyter. HackerNoon. https://hackernoon.com/working-securely-with-jupyter
4. Writer, N. N. C. (2023, December 8). Jupyter notebook ripe for cloud credential theft, researchers warn. https://www.darkreading.com/cloud-security/jupyter-notebook-cloud-credential-theft
5. Aqua Security. (2024, July 29). Real-world cyber attacks targeting data science tools. Aqua. https://www.aquasec.com/blog/cyber-attacks-data-science-tools/

## 7. Disclaimer
The content, including but not limited to code, text, images, audio, and/or video, hereafter referred to as "content", in this document are provided for informational and educational purposes only. TO THE EXTENT PERMITTED BY APPLICABLE LAW, THE AUTHOR PROVIDES THIS DOCUMENT "AS IS" WITHOUT WARRANTY OF ANY KIND, INCLUDING WITHOUT LIMITATION, ANY IMPLIED WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE, OR NONINFRINGEMENT. In no event shall the author or their employer be liable for any claim, damages or other liability, direct or indirect, whether in an action of contract, tort or otherwise, arising from, out of or in connection with the code and content or the use or other dealings in the code and content. Use this code and all other content at your own risk. 

**Third-party API Disclaimer:** Additionally, the code examples in this post may interact with third-party APIs and services. The availability and functionality of these APIs are subject to change without notice. The author is not responsible for any issues arising from changes to these APIs or any downtime or limitations imposed by the service providers. You are responsible for complying with the terms of service and usage policies of any third-party APIs you use in conjunction with this code. Use this code at your own risk, and be aware of potential security implications when connecting to external services.

**Product Link Disclaimer:** This blog post may contain links to products or services available for purchase. These links are provided to offer readers additional information and resources. The author's opinions expressed in this post are independent and not influenced by any potential commercial relationships. No compensation is received for including these links, and their presence does not constitute an endorsement. Readers are encouraged to conduct their own research before making any purchasing decisions.

## 8. Copyright
Copyright &copy; 2025 J.I. Powell. All rights reserved.
