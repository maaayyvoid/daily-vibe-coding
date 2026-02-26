

# 2026-02-26 - python wheels

## Summary
Learned the proper way to install Python packages offline using `pip download` with platform-specific flags—a huge improvement over my previous manual dependency hunting.

------



## The Problem

Since joining my current company, I've struggled with installing Python packages on an offline Kylin OS machine. The company's internal repository lacks many wheels I need.

### My Old (Painful) Workflow:

1. Download a single wheel from PyPI on my personal laptop
2. Transfer to the offline machine via USB
3. Attempt installation
4. Encounter missing dependency errors
5. Search error messages for missing packages
6. Repeat steps 1-5 **for every single dependency** 🫥

This worked for simple packages, but **matplotlib** has so many dependencies that the manual approach became unbearable—constantly switching between computers with a USB drive.😵‍💫



## Finding the Right Solution

I started by asking an LLM what wheels matplotlib requires. It gave me a list, but manually downloading each from PyPI felt inefficient. 🤨

After explaining my offline installation constraint, the LLM provided a proper 3-step workflow:

- **Step 1**: Download all wheels (on internet-connected computer)
- **Step 2**: Transfer to offline machine
- **Step 3**: Install offline



## Platform Awareness

The initial examples were Windows-focused, but I knew wheels are **platform-specific**. (Great! Something I know!🤓) My target machine is `aarch64 (ARM64) with Python 3.8`, so I requested a PowerShell version since my personal laptop runs Windows.



## Iteration & Refinement

I hit some compatibility issues (Python version mismatches), so I asked the LLM to summarize exactly what to request next time for a direct answer. 🎉

```
"Give me a one-line PowerShell command to download matplotlib and all dependencies for aarch64 Python 3.8 into a single folder, handling pure Python packages separately."
```



## Final Solution

**The one-liner that solved everything:**

```powershell
New-Item -ItemType Directory -Force -Path ".\matplotlib_aarch64_cp38" | Out-Null; pip download matplotlib --platform manylinux2014_aarch64 --python-version 38 --implementation cp --abi cp38 --only-binary=:all: -d .\matplotlib_aarch64_cp38; pip download "zipp<3.16" "importlib-resources<6.0" typing-extensions -d .\matplotlib_aarch64_cp38
```



### Key flags explained:

| Flag                               | Purpose                       |
| :--------------------------------- | :---------------------------- |
| `--platform manylinux2014_aarch64` | Target ARM64 Linux            |
| `--python-version 38`              | Python 3.8                    |
| `--implementation cp`              | CPython                       |
| `--abi cp38`                       | CPython 3.8 ABI               |
| `--only-binary=:all:`              | Wheels only, no source builds |
| `-d`                               | Destination directory         |



## Key Takeaway

**The "official" way**: Use `pip download` with explicit platform flags rather than manually chasing dependencies. For future requests, I can use this prompt template:

> *"Give me a one-line PowerShell command to download [package] and all dependencies for [platform] Python [version] into a single folder, handling pure Python packages separately."*

