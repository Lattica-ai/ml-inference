# Lattica-ai ML-inference Submission

Most relevant information regarding the model architecture and packing strategy are contained in the [public README](README.md).
Here we present only the security parameters, the execution environment, and example execution logs.


## FHE parameters and security

* **Ring dimension:** 2^12 = 4096
* **Modulus size:** 106 bits
* **Secret key distribution:** uniform ternary

These parameters achieve required 128-bit security.


## Execution environment

**Client**

* AWS c5.4xlarge
* 16 vCPUs, 32GB RAM

**Server**

* AWS EC2 g7e.2xlarge
* NVIDIA GB202 (96 GB)
* 8 vCPUs, 64 GB RAM

Both machines are in the same AWS region (us-east-1)

---

## Example execution logs

### `batch_size = 1`

```
python3 harness/run_submission.py --remote 0

20:25:31 [harness] 1: Harness: MNIST Test dataset generation completed (elapsed: 7.6382s)
20:25:34 [harness] 2.1: Communication: Get cryptographic context completed (elapsed: 2.6875s)
         [harness] Cryptographic Context size: 1.2M
20:25:36 [harness] 2.2: Client: Key Generation completed (elapsed: 2.1788s)
         [harness] Client: Public and evaluation keys size: 46.0M
20:25:39 [harness] 2.3: Communication: Upload evaluation key completed (elapsed: 3.0939s)
20:25:39 [harness] 3: Server: (Encrypted) model preprocessing completed (elapsed: 0.0002s)
20:25:42 [harness] 4: Harness: Input generation for MNIST completed (elapsed: 2.5526s)
20:25:43 [harness] 5: Client: Input preprocessing completed (elapsed: 1.5988s)
20:25:45 [harness] 6: Client: Input encryption completed (elapsed: 1.6351s)
         [harness] Client: Encrypted input size: 128.1K
20:25:47 [harness] 7: Server: Encrypted ML Inference computation completed (elapsed: 1.8457s)
         [harness] Client: Encrypted results size: 128.1K
20:25:49 [harness] 8: Client: Result decryption completed (elapsed: 1.6679s)
20:25:49 [harness] 9: Client: Result postprocessing completed (elapsed: 0.0002s)
[harness] PASS  (expected=5, got=5)
[total latency] 24.8991s
```

- Public + evaluation keys size: 46 MB
- Encrypted input size: 128 KB
- Total inference latency: 210 ms
- Compute inference latency: 22 ms
-----

### `batch_size = 100`

```
python3 harness/run_submission.py --remote 1

11:13:39 [harness] 1: Harness: MNIST Test dataset generation completed (elapsed: 7.3562s)
11:13:42 [harness] 2.1: Communication: Get cryptographic context completed (elapsed: 2.3815s)
         [harness] Cryptographic Context size: 835.0K
11:13:43 [harness] 2.2: Client: Key Generation completed (elapsed: 1.7215s)
         [harness] Client: Public and evaluation keys size: 7.3M
11:13:46 [harness] 2.3: Communication: Upload evaluation key completed (elapsed: 2.4881s)
11:13:46 [harness] 3: Server: (Encrypted) model preprocessing completed (elapsed: 0.0002s)
11:13:48 [harness] 4: Harness: Input generation for MNIST completed (elapsed: 2.6179s)
11:13:50 [harness] 5: Client: Input preprocessing completed (elapsed: 1.6294s)
11:13:53 [harness] 6: Client: Input encryption completed (elapsed: 3.0269s)
         [harness] Client: Encrypted input size: 98.0M
11:13:56 [harness] 7: Server: Encrypted ML Inference computation completed (elapsed: 2.7958s)
         [harness] Client: Encrypted results size: 1.3M
11:13:58 [harness] 8: Client: Result decryption completed (elapsed: 1.9153s)
11:13:58 [harness] 9: Client: Result postprocessing completed (elapsed: 0.0002s)
11:14:00 [harness] 10.1: Harness: Run inference for harness plaintext model completed (elapsed: 2.5667s)
[harness] Encrypted model: 0.9400 (94/100 correct)
[harness] Harness model: 0.9900 (99/100 correct)
11:14:00 [harness] 10.2: Harness: Run quality check completed (elapsed: 0.0003s)
[total latency] 28.5s
```

- Public + evaluation keys size: 7.3 MB
- Encrypted input size: 98 MB
- Total inference latency: 1.1 s
- Compute inference latency: 315 ms


