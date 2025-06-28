# Oops... (300 points) - Forensics

# **Context:**

A coder forked a repo bright,
Committed secrets late at night.
Then swift and sly, without a trace,
Deleted fork—thought safe the place.

But secrets live beyond the veil,
In logs and caches, trails prevail.
Is it gone? Or just disguised?
In CTF, the truth’s devised.

```
git push
Enumerating objects: 4, done.
Counting objects: 100% (4/4), done.
Delta compression using up to 14 threads
Compressing objects: 100% (3/3), done.
Writing objects: 100% (3/3), 393 bytes | 393.00 KiB/s, done.
Total 3 (delta 1), reused 0 (delta 0), pack-reused 0 (from 0)
remote: Resolving deltas: 100% (1/1), completed with 1 local object.
To github.com:challengehackers/newspipe2llm.git
 ac01642..391a541  main -> main
```

# Solution:

I tried accessing the forked repository but as expected from the summary it is already gone. I researched on possibilities on how to access a deleted forked repository and came upon this article. 

[https://trufflesecurity.com/blog/anyone-can-access-deleted-and-private-repo-data-github](https://trufflesecurity.com/blog/anyone-can-access-deleted-and-private-repo-data-github)

Basically, what the article states is that it is still possible to visit a deleted forked repository as long as you have the hash of the commit of the forked repository from the main branch. I searched for the main branch where a commit hash of  “ac01642” exists. With a little bit of OSINT, I was able to find the repository of the “newspipe2llm”.

![Figure 1.0 - Repository for newspipellm](Oops%20(300%20points)%20-%20Forensics%2021eb0db6ff2780c08604dde96152c47c/image.png)

Figure 1.0 - Repository for newspipellm

Checking the commits if this is truly the repository that was forked and it matches the commit hash, “ac01642”!

![Figure 1.1 - List of commit hashes](Oops%20(300%20points)%20-%20Forensics%2021eb0db6ff2780c08604dde96152c47c/image%201.png)

Figure 1.1 - List of commit hashes

![Figure 1.2 - “ac01642” commit from the main repository](Oops%20(300%20points)%20-%20Forensics%2021eb0db6ff2780c08604dde96152c47c/image%202.png)

Figure 1.2 - “ac01642” commit from the main repository

Changing the hash in the URL to “391a541”, it shows us the flag.

![Figure 1.3 - “391a541“ commit from the deleted fork](Oops%20(300%20points)%20-%20Forensics%2021eb0db6ff2780c08604dde96152c47c/image%203.png)

Figure 1.3 - “391a541“ commit from the deleted fork

The implications of being able to access deleted repositories whether private or public if a commit hash is known are possibly knowing private tokens, . This “vulnerability” in GitHub is defined as “**Cross Fork Object Reference (CFOR)”.**

References:

[https://trufflesecurity.com/blog/anyone-can-access-deleted-and-private-repo-data-github](https://trufflesecurity.com/blog/anyone-can-access-deleted-and-private-repo-data-github)

[https://trufflesecurity.com/blog/trufflehog-now-finds-all-deleted-and-private-commits-on-github](https://trufflesecurity.com/blog/trufflehog-now-finds-all-deleted-and-private-commits-on-github)

[https://www.techtarget.com/searchsecurity/news/366599096/Researcher-says-deleted-GitHub-data-can-be-accessed-forever](https://www.techtarget.com/searchsecurity/news/366599096/Researcher-says-deleted-GitHub-data-can-be-accessed-forever)

[https://medium.com/@Absolute_Z3r0/securing-deleted-github-repositories-the-surprising-risks-developers-need-to-know-76582cbb223b](https://medium.com/@Absolute_Z3r0/securing-deleted-github-repositories-the-surprising-risks-developers-need-to-know-76582cbb223b)