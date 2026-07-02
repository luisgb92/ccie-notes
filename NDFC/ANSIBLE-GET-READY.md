# Setting up the Environment

To begin, let us establish and use Python virtual environments as a standard procedure. Virtual environments allow for the installation of particular packages within an isolated setting. Tailored to your application or project.

As a result, the dependencies and packages you use don't interfere with packages of other applications or projects. For instance, you might need a specific version of a package or software development kit for your work. But another project may need a different version of that same package. This is the scenario where the advantages of virtual environments are evident.


### 1. Verify Python is installed

```python
python3 --version
```

### 2. Install the venv package (Linux only)

```python
sudo apt-get update
sudo apt-get upgrade
sudo apt install python3-venv
```

### 3. Navigate to your project directory:

```python
cd ansible_ndfc
```

### 4. Create the virtual environment:

```python
python3 -m venv venv
```

### 5. Activate the virtual environment (Linux only):

```python
source venv/bin/activate
```

When activated, your prompt will look similar to:

```python
lberista@CSCO-W-PF3RHSV8:~/ansible_ndfc$ source venv/bin/activate
(venv) lberista@CSCO-W-PF3RHSV8:~/ansible_ndfc$ 
```

### 6. To install the Requests HTTP library, run the following recommended command in your system terminal or command prompt

```python
(venv) lberista@CSCO-W-PF3RHSV8:~/ansible_ndfc$ pip install requests
Collecting requests
  Downloading requests-2.34.2-py3-none-any.whl.metadata (4.8 kB)
```