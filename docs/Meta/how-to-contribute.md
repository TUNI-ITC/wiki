# How to Contribute to This Wiki?

<!-- This wiki is backed up with the source of from [TUNI-ITC/wiki](https://github.com/TUNI-ITC/wiki). The knowledge is expected to be contributed via Markdown files (`.md`). While the style of the pages and website overall depends on [Material Theme](https://squidfunk.github.io/) for [MkDocs](https://www.mkdocs.org/). -->
This Wiki 📖 is a crowdsourced base of knowledge accumulated by generations of current and ex research assistants, students, faculty members, and other personnel. It is "whatever" annoying you don't remember after two months anymore or what frequently someone asks.

The knowledge is expected to be contributed via Markdown files (`.md`) and submitted as Pull Requests to [TUNI-ITC/wiki](https://github.com/TUNI-ITC/wiki). This guide describes this process step-by-step.

There are two options on how to contribute to Wiki:

1. Do-it-yourself 👍: you are following this guide and submitting the Pull Request.
2. Ask somebody: for instance, you are not familiar with `git` and don't want to discover its beauty, then, create an Issue in [TUNI-ITC/wiki](https://github.com/TUNI-ITC/wiki) and share a `.md` file with us. Here are the [guide](https://guides.github.com/features/mastering-markdown/) and [an online Markdown editor](https://stackedit.io/).

!!! question "What Can I Contribute?"
    Anything! Style, spelling, new knowledge, Wiki's engine. Anything, ok? 🤗

## Ok, What Should I Do?

The pipeline boils down to these several steps which should be familiar to anyone who worked with an open-source project on some sort of Hub 🤓 (GitHub, Bitbucket, GitLab, and such):

0. Create an Issue _(skip if feeling lazy or playing like a bad boy)_
1. Fork & Clone
2. Add/Change
3. Push to the Fork
4. Create a Pull Request

## 1. Create an Issue
This is completely optional and we would like to see an issue just to track the progress and maybe suggest where (which section) to put your knowledge or help you with something.

## 2. Fork & Clone
You can edit directly through the GitHub web page ("Add File" and "Edit" button) or do it locally on your computer. We recommend learning to do it the hard way by forking and cloning it manually you will benefit from this knowledge in your career, otherwise skip this step.

Fork means that you make a personal copy of this wiki - note that anyone can do that and it does not mess the main branch! To do that go to [TUNI-ITC/wiki](https://github.com/TUNI-ITC/wiki) and press the top-right button *Fork* - if you don't have a Github account yet, you need to make one.

After that, you should have the forked repo somewhere in your account, e.g., `github.com/<MY_ACCOUNT>/wiki/`.

To clone your fork locally, in terminal type:
```bash
git clone https://github.com/<MY_ACCOUNT>/wiki/
```
and it's done!

## 3. Add/Change

You may freely edit an existing file or create new, e.g., `how-to-select-a-coffee-in-a-finnish-supermarket.md`. Here are the [guide](https://guides.github.com/features/mastering-markdown/) and [an online Markdown editor](https://stackedit.io/) for you to play with.

We are using [Material Theme](https://squidfunk.github.io/) for [MkDocs](https://www.mkdocs.org/). Hence, you may also propose to add more functionality to our wiki. Check out the manuals of both to see what else we can add.

!!! info "Can I check locally how it will look?"
    Sure thing! You will need to install the `mkdocs-material` engine. It might sound overwhelming but you will need only `Python` for it (even better if you have `conda`):
    ```bash
    pip install mkdocs-material
    ```
    Once done, you can start a preview server locally
    ```bash
    cd /path/to/wiki
    mkdocs serve
    ```
    After, just type `localhost:8000` in your browser.

    See more [in the original documentation](https://squidfunk.github.io/mkdocs-material/creating-your-site/).

## 4. Push to the Changes to the Fork
Commit your changes! You know how, right?

??? info "Here is how"
    ```bash
    # it will print the modifications were made compared to last commit
    git status
    # this will `stage` you changes
    git add how-to-do-something.md
    # this will commit the staged changes
    # (it may ask you to configure git if you are doing to for the first time)
    git commit -m "added how-to-do-something.md"
    git push
    ```

## 5. Create a Pull Request
Open the page with your fork on GitHub: `github.com/<MY_ACCOUNT>/wiki/`. At this point, you should be able to find the changes you made. Somewhere at the top, you will be asked if you want to make a Pull Request and that your branch is ahead of the `main` by some commits. Make the request, by adding comments, title, and check that you are proposing the files you expect and submit it. Someone will review and accept it. That's it!
