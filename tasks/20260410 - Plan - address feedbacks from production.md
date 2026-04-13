# Implementation Plan: Feedback from Production

Below are raw notes with feedbacks and observations users encountered in production while using Spec-to-code.

1. When working on large featiures SDLC documebtation structure is not adequate. vision, spec, requirements documents are sufficient to establish baseline and decribe initial capabilities. But when large feature may require it's own problem statement, spec, architecture and implementation plan. SDLC must reflect this and explain how to manage feature-releted documents.

2. Vision is only applicable for initial product discovery. Once done, we need different document to describe feature-level additional capabilities. The role of the document is to explain motivation, business case and/or use cases. But it can't be a 'vision' because vision is a single document that describes the product in a broad sense.

3. 'spec` and `requirements` documents look similar and are not sufficiently different. They should be more clearly separated and have different roles. Unless you have compelling argument why we need both, we can combine them into a single document. The purpose of such combined document is to provide non-ambiguos details about desired functionality. It serves as a contract between product owner and development team and provides input for design and implementation.

4. Define location for short-lived 'implementation plan' documebt. Currently it is created in '/docs' which is intended for permanent documemtation. Better place would be under './tasks' while plan is ready or active, and then move to 'done' along with tasks it contanins.  