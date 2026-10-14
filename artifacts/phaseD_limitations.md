## Limitations


1. **Real vibration never reached Zone C or D.** All 1 hp conditions lay in ISO Zone A (0.14–0.29 mm/s against an A/B boundary of 0.71 mm/s; see Step 31b for 2 hp), and velocity did not increase consistently with fault size. The derating and stopping behaviour was therefore demonstrated by fault injection on real recordings (Step 31c), not by recordings of genuinely dangerous vibration. Scenario A is a severity scaling, not a physical fault model.

2. **Load sensitivity is systematic, not random.** 27.9% of correctly classified 2 hp windows were flagged, and aggregation does not remove it (27.9% at K = 1, 23.1% at K = 20). After a load change the system would route many correct diagnoses to inspection. Training on several loads is the natural remedy.

3. **Small test set.** About 46 test windows per known class (320 in total). Zero errors bound the true error rate only to below 1.1% (95%), and the known-fault flag rate of 5.6% has a 95% interval of 3.4%–8.7%.

4. **Seed sensitivity of the novelty score.** Unseen-fault detection varied between training runs (87.9 ± 12.7%, range 65.7%–98.1%). A deep ensemble reached 96.8%. Machine safety was unaffected, because the physics layer does not depend on the network.

5. **Scope of the standard.** The CWRU motor (≈ 1.5 kW) is below the 15 kW scope of ISO 10816-3; ISO 10816-1 Class I values were used as engineering reference values, not as a certified acceptance criterion.

6. **Units and measurement.** CWRU acceleration was assumed to be in g, the usual assumption for this dataset. One magnetically mounted accelerometer at 12 o'clock was used, whereas ISO evaluates several directions.

7. **Theory.** The capacity rule is CBF-inspired, not a formal Control Barrier Function: there is no dynamic machine model and no forward-invariance proof. The policy evaluation uses the ISO boundary as the safety truth.

8. **Single dataset and rig.** All results come from one laboratory test rig with seeded (EDM) faults at 1 hp and 2 hp.