# Lattica-ai ML-inference Submission

Most relevant information regarding the model architecture and packing strategy are contained in the [public README](README.md).
Here we present only the security parameters, the execution environment, and example execution logs.


## FHE parameters and security

* **Ring dimension:** 2^13 = 8192
* **Modulus size:** 122 bits
* **Secret key distribution:** uniform ternary

These parameters are well above required 128-bit security.


## Execution environment

**Client**

* AWS c5.4xlarge
* 16 vCPUs, 32GB RAM

**Server**

* AWS EC2 g6e.2xlarge
* NVIDIA L40S (48 GB)
* 8 vCPUs, 64 GB RAM

Both machines are in the same AWS region (us-east-1)

---

## Example execution logs

### `batch_size = 1`

```
python3 harness/run_submission.py --remote 0

14:57:03 [harness] 1: Test dataset generation completed (elapsed: 7.319s)
14:57:06 [harness] 2.1: Communication: Get cryptographic context completed (elapsed: 2.4785s)
         [harness] Cryptographic Context size: 2.4M
14:57:09 [harness] 2.2: Key Generation completed (elapsed: 3.4212s)
         [harness] Public and evaluation keys size: 113.0M
14:57:14 [harness] 2.3: Communication: Upload evaluation key completed (elapsed: 4.883s)
14:57:14 [harness] 3: Encrypted model preprocessing completed (elapsed: 0.0002s)
14:57:17 [harness] 4: Input generation completed (elapsed: 2.564s)
14:57:18 [harness] 5: Input preprocessing completed (elapsed: 1.6013s)
14:57:20 [harness] 6: Input encryption completed (elapsed: 1.6352s)
         [harness] Encrypted input size: 256.1K
14:57:22 [harness] 7: Encrypted computation completed (elapsed: 1.8769s)
         [harness] Encrypted results size: 256.1K
14:57:23 [harness] 8: Result decryption completed (elapsed: 1.6911s)
14:57:23 [harness] 9: Result postprocessing completed (elapsed: 0.0002s)
[harness] PASS  (expected=5, got=5)
         [submission] Server reported steps: {'Encrypted computation': 0.048, 'Backend overhead': 0.098, 'Upload time': 0.013, 'Download time': 0.065}
         [submission] Encrypted computation: 0.048s
         [submission] Backend overhead: 0.098s
         [submission] Upload time: 0.013s
         [submission] Download time: 0.065s
[total latency] 27.4704s
```

- Public + evaluation keys size: ~113 MB
- Encrypted input size: ~256 KB
- Total inference latency: ~2 s
- Compute inference latency: ~50 ms
-----

### `batch_size = 100`

```
python3 harness/run_submission.py --remote 1

15:05:47 [harness] 1: Test dataset generation completed (elapsed: 7.0674s)
15:05:48 [harness] 2.1: Communication: Get cryptographic context completed (elapsed: 1.9238s)
         [harness] Cryptographic Context size: 1.6M
15:05:50 [harness] 2.2: Key Generation completed (elapsed: 1.8694s)
         [harness] Public and evaluation keys size: 16.5M
15:05:53 [harness] 2.3: Communication: Upload evaluation key completed (elapsed: 2.4426s)
15:05:53 [harness] 3: Encrypted model preprocessing completed (elapsed: 0.0002s)
15:05:55 [harness] 4: Input generation completed (elapsed: 2.6099s)
15:05:57 [harness] 5: Input preprocessing completed (elapsed: 1.6371s)
15:06:02 [harness] 6: Input encryption completed (elapsed: 4.7105s)
         [harness] Encrypted input size: 196.0M
15:06:06 [harness] 7: Encrypted computation completed (elapsed: 4.0873s)
         [harness] Encrypted results size: 2.5M
15:06:08 [harness] 8: Result decryption completed (elapsed: 2.2307s)
15:06:08 [harness] 9: Result postprocessing completed (elapsed: 0.0002s)
15:06:11 [harness] 10.1: Harness: Run inference for harness plaintext model completed (elapsed: 2.5742s)
[harness] Encrypted model: 0.9400 (94/100 correct)
[harness] Harness model: 0.9900 (99/100 correct)
15:06:11 [harness] 10.2: Harness: Run quality check completed (elapsed: 0.0003s)
         [submission] Server reported steps: {'Encrypted computation': 0.628, 'Backend overhead': 0.081, 'Upload time': 1.494, 'Download time': 0.102}
         [submission] Encrypted computation: 0.628s
         [submission] Backend overhead: 0.081s
         [submission] Upload time: 1.494s
         [submission] Download time: 0.102s
[total latency] 31.1536s
```

- Public + evaluation keys size: ~16 MB
- Encrypted input size: ~196 MB
- Total inference latency: ~4 s
- Compute inference latency: 650 ms


