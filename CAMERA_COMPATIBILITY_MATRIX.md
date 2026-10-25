# Camera compatibility matrix

The product aims for broad compatibility with existing IP cameras and NVRs. It cannot promise that every CCTV device will connect: access, codecs, firmware, network topology, and vendor behavior differ. Qualify an exact camera/NVR model and firmware before calling it supported.

## Qualification states

| State | Evidence required |
| --- | --- |
| Not assessed | Model or firmware not confirmed. |
| Protocol listed | Device settings or vendor documentation indicate RTSP/ONVIF support; no stream test yet. |
| Stream validated | The restaurant-side gateway can access and decode the intended stream using the chosen credential-handling path. |
| Pilot qualified | Reconnect, stale-feed, latency, image quality, zone mapping, and privacy/retention checks pass at the pilot site. |
| Not supported | No approved accessible stream path, or required reliability/privacy controls cannot be met. |

Protocol presence alone is never a compatibility result. Record the evidence and test date for every qualification change.

## What protocol labels do and do not prove

- **RTSP:** A device listing RTSP does not establish the RTSP version, media transport, codec, authentication behavior, or whether a client on the restaurant gateway can reach the stream. RTSP 2.0 (RFC 7826) explicitly is not backward compatible with RTSP 1.0, so a generic “RTSP supported” note is not enough to select an adapter. Record the observed version when available, TCP/UDP transport, codec/profile, and an actual gateway decode/reconnect result. See the [RFC Editor's RTSP 2.0 record](https://www.rfc-editor.org/info/rfc7826/).
- **ONVIF:** Check the exact product and firmware in the [ONVIF Conformant Products database](https://www.onvif.org/conformant-products/). ONVIF states that interoperability conformance applies to registered products and the listed firmware/software version; a menu setting or vendor logo alone is not proof.
- **Profile T:** This is ONVIF's advanced video streaming profile, including H.264/H.265 and HTTPS streaming features. Some capabilities are conditional, so verify the exact feature and stream on the pilot device rather than assuming every Profile T product exposes the same options. See [ONVIF Profile T](https://www.onvif.org/profiles/profile-t/).
- **Profile S:** Existing Profile S products may continue basic streaming, but ONVIF will stop accepting new Profile S conformance submissions for new products or firmware/software versions after **March 31, 2027**. ONVIF also recommends moving away from the Profile S username-token authentication method toward more secure supported mechanisms. Record the authentication mode and include it in the security review; do not enable or change it just to qualify a device. See [ONVIF's Profile S deprecation Q&A](https://www.onvif.org/profiles/profile-s/profile-s-deprecation-qna/).

These standards improve the odds of broad camera support, but they do not make every CCTV camera, NVR, network path, codec, or firmware compatible. The product must qualify each exact pilot device and retain a tested-adapter matrix.

## Device test record

Copy one record per camera/NVR and stream profile. Do not include passwords, secret-bearing stream URLs, public IP addresses, or customer footage in this worksheet.

| Field | Value |
| --- | --- |
| Restaurant / site reference |  |
| Recorder vendor and model |  |
| Recorder firmware |  |
| Camera vendor and model (if different) |  |
| Camera firmware |  |
| Dining-area camera count |  |
| RTSP listed/enabled (yes/no/unknown) |  |
| ONVIF listed/enabled and profile (if shown) |  |
| Codec and stream profile (validated, not guessed) |  |
| Resolution / frame rate |  |
| Authentication mode (describe type only) |  |
| Gateway network path and required firewall rules |  |
| Qualification state |  |
| Evidence reference and test date |  |

## Pilot checks

- Confirm the selected stream is the dining-area view and tables remain visible through normal lighting, crowding, and occlusion.
- Validate decode behavior, end-to-end frame age, reconnect after a network interruption, and stale/offline transitions.
- Measure resolution, frame rate, and processing load at the intended gateway settings.
- Map table polygons/zones with restaurant staff and record the mapping version and camera viewpoint.
- Verify that table-level occupancy and dwell output does not identify people; document approved signage, purpose, retention, and access.
- Keep low-confidence output as a review suggestion. Test staff correction and audit history before considering any automatic notification.
- Recheck the exact device model, firmware, stream profile, and network path after a firmware or configuration change.

## Integration boundary

Prefer a restaurant-side gateway with adapters for common standard protocols. Keep camera credentials on that gateway, outside browser code and client-visible configuration. Add vendor-specific adapters only for a named pilot device and only after the security and privacy review. A device without an approved, accessible stream path remains unsupported until a validated adapter exists.
