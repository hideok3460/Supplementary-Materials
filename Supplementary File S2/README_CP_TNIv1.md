# Same-backend CP-TNI supplementary files

`verification_canonical_noncanonical_CP_TNI.ipynb` is the executed analysis notebook for the single combined IBM Quantum experiment on `ibm_phoenix` (job `daj7e68mhr3c73e8kotg`). It contains 216 circuits at 4096 shots each, coherence-sensitive state tomography for the canonical `(M1,M2)=(0,0)` and phase-sensitive `(0,1)` resource members, four-probe logical tomography, convex CP-TNI Choi fitting, and 500-sample multinomial bootstrap intervals.

`verification_canonical_noncanonical_CP_TNI_reproducibility_export.ipynb` is the executed exporter/validator for the hardware provenance package. `Exact_Channel_Equivalence_hardware_reproducibility_archive.zip` contains the actual hardware artifacts from the same job: 216 per-circuit raw-count JSON files, a consolidated ordered raw-count file, a 216-row final-layout CSV, a contemporaneous backend calibration snapshot, provenance and manifest records, an English README, and SHA-256 checksums. The archive records 4096 shots for every circuit (884,736 total shots). The calibration snapshot precedes job submission by 11.860122 seconds. Manifest indices, circuit names, per-circuit file order, and final-layout order agree for all 216 circuits; no final layout is missing. The ZIP contains 223 entries, all 222 entries listed in `SHA256SUMS.txt` pass SHA-256 verification, and the ZIP CRC check passes.

The process reconstructions use preselected representative sectors: `(x,m)=(1,0)` for the canonical resource, which exercises the nontrivial `X ⊗ I` frame, and `(x,m)=(0,0)` for the phase-sensitive resource, which exercises the `I ⊗ XZ` frame. Because both resource and sector differ, their numerical process metrics are not a controlled resource-performance comparison.

The supplied CSV files reproduce the central state and process summaries printed by the executed notebook. The notebook fixes the base random seed at `240917`, uses percentile bootstrap intervals, and the bootstrap intervals quantify finite-shot uncertainty for the single submitted batch, not run-to-run drift, calibration uncertainty, or SPAM uncertainty. No error mitigation or SPAM separation was applied.

The reproducibility exporter notebook and complete hardware archive are also publicly released in the companion GitHub repository cited as `okazaki2` in the main manuscript. This makes the raw data and provenance independent of future IBM Quantum job-retention or account-access constraints.


## Revision v1: tomography PSD-projection audit and process-metric interpretation

The state reconstruction used by the executed analysis notebook is more specific than a generic
"PSD projection." After Pauli linear inversion, the matrix is Hermitized and normalized to unit
trace. Its eigenvalues are then projected onto the probability simplex by
`lambda'_i = max(lambda_i - tau, 0)`, with `tau` chosen so that the projected eigenvalues sum to
one. Recombining the unchanged eigenvectors gives the Frobenius-nearest positive-semidefinite,
trace-one matrix. This is an eigenvalue-simplex projection; it is not naive negative-eigenvalue
clipping followed by renormalization and it does not use CVXPY.

An independent audit from the released per-circuit raw counts gives the following results.

- All 16 point-estimate state-tomography linear inversions were already PSD. Their minimum
  eigenvalues range from 7.71e-4 to 2.347e-2.
- All eight point-estimate receiver matrices used for the four-probe process reconstruction were
  also already PSD, with minimum eigenvalues from 4.575e-3 to 2.141e-2.
- Hence the projection changed none of the 24 point estimates beyond floating-point roundoff:
  maximum Frobenius shift 9.95e-16, maximum state-fidelity shift below 9e-16, and maximum
  complex code-coherence shift 2.01e-16.
- The same projection is applied to every bootstrap reconstruction. Among 8,000 state-tomography
  bootstrap matrices, 561 (7.01%) required a nontrivial projection. Among 4,000 process-probe
  bootstrap receiver matrices, 304 (7.60%) required one.
- For recovered-state bootstrap matrices, the projection changed fidelity by 2.07e-5 on average
  over all resamples (6.43e-4 among active projections; maximum 2.58e-3) and changed the complex
  coherence by 1.04e-5 on average (3.21e-4 among active projections; maximum 1.85e-3).

Because the PSD projection is nonlinear, it can introduce boundary bias when a finite-shot linear
estimate leaves the state set. The revised manuscript therefore does not describe the projected
state estimator as unbiased. Instead it reports that the point estimates are unaffected, applies the
same reconstruction to every bootstrap resample, and treats the bootstrap intervals as conditional
finite-shot uncertainty under that fixed reconstruction pipeline.

The process analysis does **not** use separately normalized code-space probe outputs. Each logical
block is retained with its measured trace and all four blocks are fit simultaneously to one linear
CP-TNI Choi matrix. Accordingly, the reported PTM is a linear PTM of a trace-non-increasing
code-space suboperation, not a conditional response matrix and not generally a CPTP PTM. The
survival-weighted identity overlap is not a standard CPTP process fidelity or average gate fidelity.
Leakage is retained explicitly through the mean code-space survival, which is 0.9053 for the
canonical representative sector and 0.9506 for the phase-sensitive representative sector in this
single batch. The conditional diagnostics `D_id^cond = 0.9470` and `0.9647` described in the
review comment are not used by the present CP-TNI analysis.

`hardware_same_backend_state_summaryv1.csv` adds pre/post-projection point-estimate diagnostics.
`hardware_same_backend_cptni_process_summaryv1.csv` adds machine-readable fields specifying
that the process map is linear CP-TNI, uses unnormalized code blocks, and retains leakage.
