# Part 6 7-Level Photon-Mapping Notes

Files in this folder use the finalized experiment matrix:
- LowN: N=16384, radius=0.10
- MidN: N=65536, radius=0.10
- HighN: N=196608, radius=0.10
- HighN LowRad: N=196608, radius=0.10/sqrt(3) = 0.057735
- SuperHighN: N=393216, radius=0.10
- SuperHighN LowRad: N=393216, radius=0.10/sqrt(3) = 0.057735
- SuperHighN SuperLowRad: N=393216, radius=0.10/sqrt(6) = 0.040825
- Bonus bunny only: N=983040, radius=0.05, kd-tree only

Column definitions:
- Stored Photons: photons kept in the photon map after build-pass filtering.
- Hit Rate: gather_hits / gather_queries, i.e. the fraction of gather queries that returned at least one accepted photon.
- Avg. Photons / Query: accepted photons averaged over all gather queries.
- Avg. Photons / Hit: accepted photons averaged only over successful gather queries.
- Max in Query: maximum accepted photons observed in any single gather query.
- Fallback Indirect: number of indirect-light evaluations in which recursive path tracing still participated. This includes hard fallback and blended cases, so it is not a pure failure counter.

Main visual findings:
- Moving from MidN to HighN at the original radius makes the photon estimator much denser: Avg. Photons / Query rises from about 3.5-3.9 to about 10-11, and the wall artifacts become much stronger and more obvious.
- Reducing the radius to 0.10/sqrt(3) at HighN brings the gather density back down near the MidN range (roughly 3.8-4.1 Avg. Photons / Query), which visually suppresses many of the oversized wall discs.
- SuperHighN at the original radius is the strongest photon-driven regime. Avg. Photons / Query rises to about 16-17 and Max in Query saturates at 24 in all three scenes. This produces the most aggressive translucent disc artifacts on diffuse walls.
- SuperHighN LowRad still keeps a fairly dense photon estimate (about 7.3-7.5 Avg. Photons / Query), so the photon map remains strong, but the artifacts are more controlled than in SuperHighN with radius 0.10.
- SuperHighN SuperLowRad pushes the estimator back down near about 3.8 Avg. Photons / Query, very close to MidN / HighN LowRad. Visually this reduces the worst over-smoothing and large circular artifacts, but it also weakens the photon contribution.
- The kd-tree does not materially change image appearance for matched parameters, but it consistently makes the dense settings far more practical. The speed gap becomes especially large in HighN and SuperHighN configurations, where linear scan spends much more time on nearby-photon searches.
- In CBdragon, the mirror dragon itself is still handled by the original recursive path tracer, but the diffuse Cornell Box walls still show the same photon-density trends as the other scenes. This is useful evidence that the current implementation mainly changes diffuse indirect illumination on the environment.
- The bonus bunny run (15x photons, radius 0.05, kd-tree) still shows strong photon-map influence. Raising the photon count alone does not remove the kernel-shaped wall artifacts when the radius remains large enough to collect many photons; it mostly makes the estimator denser and more dominant.
