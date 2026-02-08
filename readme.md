Discover meetings endpoints and recording flow

Searched for regex `Meeting|Meetings|meeting` (`**/*.cs`), 200 results

Searched for regex `recording|ffmpeg|S3|AmazonS3|bucket|presign` (`**/*.*`), 200 results

Read [](file:///c%3A/Users/PC%20Rent/Desktop/CustomerService/N5-Customer-Service/N5.CustomerService.Api/Controllers/MeetingsController.cs)

Read [](file:///c%3A/Users/PC%20Rent/Desktop/CustomerService/N5-Customer-Service/N5.CustomerService.Api/Application/Commands/Meetings/StartUpload/StartUploadCommandHandler.cs)

Read [](file:///c%3A/Users/PC%20Rent/Desktop/CustomerService/N5-Customer-Service/N5.CustomerService.Api/Application/Commands/Meetings/UploadPart/UploadPartCommandHandler.cs)

Read [](file:///c%3A/Users/PC%20Rent/Desktop/CustomerService/N5-Customer-Service/N5.CustomerService.Api/Application/Commands/Meetings/CompleteUpload/CompleteUploadCommandHandler.cs)

Read [](file:///c%3A/Users/PC%20Rent/Desktop/CustomerService/N5-Customer-Service/N5.CustomerService.Api/Application/Commands/Meetings/RequestTranscriptGeneration/RequestTranscriptGenerationCommandHandler.cs)

Read [](file:///c%3A/Users/PC%20Rent/Desktop/CustomerService/N5-Customer-Service/N5.CustomerService.Api/Application/Commands/Meetings/UpsertMeetingMetadata/UpsertMeetingMetadataCommandHandler.cs)

Read [](file:///c%3A/Users/PC%20Rent/Desktop/CustomerService/N5-Customer-Service/N5.CustomerService.Api/Application/Queries/Meetings/GetMeetingsByCustomer/GetMeetingsByCustomerQueryHandler.cs)

Read [](file:///c%3A/Users/PC%20Rent/Desktop/CustomerService/N5-Customer-Service/N5.CustomerService.Domain/AggregatesModel/MeetingAggregate/Meeting.cs)

Read [](file:///c%3A/Users/PC%20Rent/Desktop/CustomerService/N5-Customer-Service/N5.CustomerService.Domain/AggregatesModel/MeetingAggregate/MeetingMetadata.cs)

Read [](file:///c%3A/Users/PC%20Rent/Desktop/CustomerService/N5-Customer-Service/N5.CustomerService.Infrastructure/Repository/MeetingsRepository.cs)

Read [](file:///c%3A/Users/PC%20Rent/Desktop/CustomerService/N5-Customer-Service/N5.CustomerService.Api/Application/Commands/Meetings/UpdateMeetingAnalysis/UpdateMeetingAnalysisCommandHandler.cs)

Read [](file:///c%3A/Users/PC%20Rent/Desktop/CustomerService/N5-Customer-Service/N5.CustomerService.Api/Application/Queries/Meetings/GetMeetingSummary/GetMeetingSummaryQueryHandler.cs)

Read [](file:///c%3A/Users/PC%20Rent/Desktop/CustomerService/N5-Customer-Service/N5.CustomerService.Api/Application/Queries/Meetings/GetProcessedMeetingsWithAnalysis/GetProcessedMeetingsWithAnalysisQueryHandler.cs)

Read [](file:///c%3A/Users/PC%20Rent/Desktop/CustomerService/N5-Customer-Service/N5.CustomerService.Api/Infrastructure/Services/Implementation/TranscriptGenerationService.cs)

Read [](file:///c%3A/Users/PC%20Rent/Desktop/CustomerService/N5-Customer-Service/N5.CustomerService.Api/Application/Queries/Meetings/GetMeetingTranscriptTxt/GetMeetingTranscriptTxtQueryHandler.cs)

Searched for regex `python|ffmpeg|transcript service|fluentd|microservice|analysis service|TranscriptService` (`**/*.cs`), 52 results

Read [](file:///c%3A/Users/PC%20Rent/Desktop/CustomerService/N5-Customer-Service/N5.CustomerService.Api/Infrastructure/Services/Implementation/TranscriptService.cs)

Read [](file:///c%3A/Users/PC%20Rent/Desktop/CustomerService/N5-Customer-Service/N5.CustomerService.Api/Application/Events/ExternalEvents/TranscriptResultExternalEventHandler.cs)

Read [](file:///c%3A/Users/PC%20Rent/Desktop/CustomerService/N5-Customer-Service/N5.CustomerService.Domain/AggregatesModel/MeetingAggregate/MeetingFactory.cs)

Read [](file:///c%3A/Users/PC%20Rent/Desktop/CustomerService/N5-Customer-Service/N5.CustomerService.Domain/AggregatesModel/MeetingAggregate/MeetingAnalysis.cs)

Read [](file:///c%3A/Users/PC%20Rent/Desktop/CustomerService/N5-Customer-Service/N5.CustomerService.Domain/AggregatesModel/MeetingAggregate/MeetingUploadPart.cs)

Read [](file:///c%3A/Users/PC%20Rent/Desktop/CustomerService/N5-Customer-Service/N5.CustomerService.Domain/AggregatesModel/GroupNodeAggregate/IMeetingsRepository.cs)

Read [](file:///c%3A/Users/PC%20Rent/Desktop/CustomerService/N5-Customer-Service/N5.CustomerService.Infrastructure/Repository/MeetingAnalysisRepository.cs)

Read [](file:///c%3A/Users/PC%20Rent/Desktop/CustomerService/N5-Customer-Service/N5.CustomerService.Domain/AggregatesModel/MeetingAggregate/IMeetingAnalysisRepository.cs)

Read [](file:///c%3A/Users/PC%20Rent/Desktop/CustomerService/N5-Customer-Service/N5.CustomerService.Domain/AggregatesModel/MeetingAggregate/MeetingState.cs)

Searched for regex `ffmpeg|FFmpeg|python`, 1 result

Read [](file:///c%3A/Users/PC%20Rent/Desktop/CustomerService/N5-Customer-Service/N5.CustomerService.Infrastructure/Finder/MeetingsFinder.cs)

Read [](file:///c%3A/Users/PC%20Rent/Desktop/CustomerService/N5-Customer-Service/N5.CustomerService.Api/Configuration/appsettings.json#180-180), lines 180 to 226

## Plan: Segmented Recording Refactor (DRAFT)

Introduce a new segmented-recording flow alongside the existing multipart flow, scoped strictly to backend changes. Keep legacy upload/transcript endpoints working; add a parallel segment-based set. Assume `/complete` always arrives (no timers). Route all new uploads directly to S3 under `{userId}/{meetingId}/`. Do not add frontend or Python changes; just call the existing external orchestrator when needed.

**Steps**
1. Survey existing meeting endpoints and handlers in MeetingsController.cs to mark which remain untouched (legacy multipart upload, transcript retrieval/request, analysis update, metadata upsert, meeting reads) and where to add the new segmented endpoints without changing current routes.
2. Extend the domain with a `RecordingSegment` value/object and optional meeting fields for recording job/result in Meeting.cs; define persistence mapping in EntityConfigurations and add a migration for the new table/columns.
3. Expand repository contracts in N5.CustomerService.Domain/Interfaces/IMeetingsRepository.cs for segment upsert/list and job/result updates; implement SQL/storage logic in MeetingsRepository.cs, preserving legacy part logic.
4. Add a Python orchestrator client interface/implementation under Infrastructure (e.g., Services), configure settings in appsettings.json, and wire DI in Startup.cs. The client only sends meetingId + ordered segments + callbackUrl; no local FFmpeg work.
5. Introduce MediatR commands/handlers in Meetings:
   - StartRecording: create meeting (or reuse existing) and return identifiers; set base S3 prefix to `{userId}/{meetingId}/`.
   - UploadSegment: validate meeting exists, SegmentIndex > 0, SizeBytes > 0; upload directly to S3 at the prefix; upsert segment metadata.
   - CompleteRecording: fetch ordered segments, invoke orchestrator client, persist jobId/state; do not auto-complete on timeouts.
   - RecordingCallback: receive completion from orchestrator, persist final file key/status, move meeting state to Processed/Failed.
   Keep existing multipart handlers intact.
6. Expose new endpoints in MeetingsController.cs parallel to legacy ones (e.g., `/meetings/start-recording`, `/meetings/upload-segment`, `/meetings/complete-recording`, `/meetings/{meetingId}/recording-complete-callback`), leaving legacy multipart routes unchanged.
7. Enforce validation/idempotency: guard duplicate `(MeetingId, SegmentIndex)` inserts, ensure callback authentication, and avoid fetching remote S3 resources beyond existing transcript reads. No auto-`/complete` fallback timers.
8. Update storage path usage: ensure new handlers and any persisted keys use `{userId}/{meetingId}/` prefixes; leave legacy transcript/analysis paths untouched unless explicitly in scope.
9. Add tests: unit tests for segment upload/complete logic and repository segment persistence; integration test for segmented flow (upload segments → complete → callback → state updated).

**Verification**
- Run `dotnet test` across the solution.
- Manual API checks for new routes: start-recording → upload-segment → complete-recording → callback; confirm S3 keys follow `{userId}/{meetingId}/` and meeting state transitions correctly.

**Decisions**
- Backend-only scope; frontend and Python service are out of scope.
- Assume `/complete` is always delivered; no timeout-driven completion.
- Legacy multipart endpoints remain; segmented flow is additive.
- New uploads write directly to S3 under `{userId}/{meetingId}/`; no new remote S3 fetches.
