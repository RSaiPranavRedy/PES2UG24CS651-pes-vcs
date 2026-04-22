# PES-VCS Lab Report

**Name:** Sai Pranav Reddy  
**SRN:** PES2UG24CS651  
**GitHub Repository:** [https://github.com/SaiPranavRedy/PES2UG24CS651-pes-vcs](https://github.com/SaiPranavRedy/PES2UG24CS651-pes-vcs)

---

## Phase 1: Object Storage Foundation

In this phase, I implemented content-addressable object storage using SHA-256 hashing.  
Objects are stored inside `.pes/objects` using sharded directories, and object integrity is verified during reads.

### Screenshot 1A: `./test_objects` Output
<img width="1235" height="260" alt="test_object" src="https://github.com/user-attachments/assets/e5290eb7-7c3c-4200-ab66-9f2452874278" />
![Screenshot 1A](screenshots/test_objects.png)

### Screenshot 1B: `.pes/objects` Directory Structure

![Screenshot 1B](screenshots/pes-objects.png)
<img width="1159" height="120" alt="Screenshot 2026-04-21 at 11 56 43 PM" src="https://github.com/user-attachments/assets/11624b7b-66de-4a34-a776-a91ba7e8c952" />

---

## Phase 2: Tree Objects

In this phase, I implemented tree construction from the index to represent directory snapshots.  
Tree objects store file and directory entries using modes, names, and object hashes.

### Screenshot 2A: `./test_tree` Output

![Screenshot 2A](screenshots/test_tree.png)
<img width="921" height="174" alt="Screenshot 2026-04-21 at 11 57 16 PM" src="https://github.com/user-attachments/assets/c99563e6-6035-47dc-a745-ece69eb40403" />

### Screenshot 2B: Raw Object Using `xxd`

![Screenshot 2B](screenshots/xxd.png)
<img width="932" height="574" alt="Screenshot 2026-04-22 at 12 02 04 AM" src="https://github.com/user-attachments/assets/7f0ed09d-d71c-4b2b-9458-9803fba9fa99" />

---

## Phase 3: Index / Staging Area

In this phase, I implemented index loading, saving, and file staging.  
The index stores staged file metadata including mode, blob hash, modification time, size, and path.

### Screenshot 3A: `pes init`, `pes add`, and `pes status`
<img width="907" height="578" alt="Screenshot 2026-04-21 at 11 59 11 PM" src="https://github.com/user-attachments/assets/ac7d43bc-13d0-4152-837b-69b376d09ad9" />

![Screenshot 3A](screenshots/3A.png)

### Screenshot 3B: `.pes/index` Contents

![Screenshot 3B](screenshots/pes-index.png)
<img width="847" height="61" alt="Screenshot 2026-04-21 at 11 59 27 PM" src="https://github.com/user-attachments/assets/88f7dba2-3b94-4446-9da0-091b1fd64867" />

---

## Phase 4: Commits and History

In this phase, I implemented commit creation using the staged index and generated root tree object.  
Each commit stores tree hash, parent commit, author, timestamp, and commit message.

### Screenshot 4A: `./pes log` Output
<img width="783" height="323" alt="Screenshot 2026-04-22 at 12 00 01 AM" src="https://github.com/user-attachments/assets/7d5b60d4-940a-4e82-8f19-fb4507ab17b7" />

![Screenshot 4A](screenshots/pes-logs.png)

### Screenshot 4B: `.pes` Files After Three Commits
<img width="791" height="816" alt="Screenshot 2026-04-22 at 12 01 37 AM" src="https://github.com/user-attachments/assets/28b7ed1b-5961-48c8-ad99-b7ddf3aa37e2" />
<img width="820" height="621" alt="Screenshot 2026-04-22 at 12 01 50 AM" src="https://github.com/user-attachments/assets/66832191-8bef-4c18-a0bd-c42c46e1ba38" />
<img width="791" height="816" alt="Screenshot 2026-04-22 at 12 01 37 AM" src="https://github.com/user-attachments/assets/52873687-e395-4bd6-b2ed-99ae86947f21" />
<img width="700" height="53" alt="Screenshot 2026-04-22 at 12 00 27 AM" src="https://github.com/user-attachments/assets/72a07762-cf48-4d66-91b2-3877b28c7bde" />
<img width="782" height="238" alt="Screenshot 2026-04-22 at 12 00 09 AM" src="https://github.com/user-attachments/assets/7bab9bfe-ff64-4710-be67-9919a8544ad6" />

![Screenshot 4B](screenshots/4B.png)

### Screenshot 4C: HEAD and Branch Reference

![Screenshot 4C](screenshots/4C.png)

---

## Final Integration Test

The final integration test verifies repository initialization, staging, committing, logging, reference updates, and object store creation.

### Screenshot: `make test-integration`


<img width="791" height="816" alt="Screenshot 2026-04-22 at 12 01 37 AM" src="https://github.com/user-attachments/assets/8202bfe3-23f2-4f24-b7cb-dae3dc98ff65" />

![Final Integration Test Part 1](screenshots/test-inte1.png)


<img width="820" height="621" alt="Screenshot 2026-04-22 at 12 01 50 AM" src="https://github.com/user-attachments/assets/f6055f2f-afe7-4780-b2d8-30d9935c5622" />

![Final Integration Test Part 2](screenshots/test-inte2.png)

---

## Analysis Questions

### Q5.1: Implementing `pes checkout <branch>`

`pes checkout <branch>` would first check whether `.pes/refs/heads/<branch>` exists. If it exists, `.pes/HEAD` would be updated to contain `ref: refs/heads/<branch>`.

After updating HEAD, PES-VCS would read the commit hash from that branch, load the commit object, load its root tree, and update the working directory to match that tree snapshot.

### Q5.2: Dirty Working Directory Detection

A dirty working directory conflict can be detected by comparing tracked files in the working directory with the index. PES-VCS can check file size, modification time, and, if needed, rehash file contents and compare them with the stored blob hash.

If a tracked file has uncommitted changes and the target branch contains a different version of that file, checkout should refuse to continue to avoid overwriting user work.

### Q5.3: Detached HEAD

Detached HEAD means `.pes/HEAD` stores a commit hash directly instead of pointing to a branch reference. New commits can still be created, but no branch name automatically points to them.

The user can recover those commits by creating a new branch file inside `.pes/refs/heads/` containing the detached commit hash, or by updating an existing branch to point to that commit.

### Q6.1: Garbage Collection

Garbage collection would start from all branch references in `.pes/refs/heads/` and mark every reachable commit. From each commit, it would recursively visit its tree objects, subtrees, and blob objects.

All reachable hashes can be stored in a hash set. After that, PES-VCS can scan `.pes/objects` and delete objects whose hashes are not present in the reachable set.

### Q6.2: Concurrent Garbage Collection Risk

Garbage collection is dangerous during a commit because the commit process may write new objects before updating the branch reference. During that gap, GC may think the new objects are unreachable and delete them.

Git avoids this type of race using locking, temporary object protection, and grace periods before deleting unreachable objects.
