# AI participant in a multi-party phone conference that speaks only when addressed by name, joined through a single-use ticket, with per-leg consent

**Defensive Publication** - published to establish prior art.
**Publisher:** Gus IT LLC.

## Abstract

A design for adding an AI assistant to a multi-party telephone conference without it talking over the humans. The assistant joins as its own conference leg whose audio is streamed to a realtime speech bridge; the bridge identifies the assistant and conference from a single-use join ticket bound to both and consumed when the stream starts. The realtime session uses server-side voice activity detection with automatic response creation disabled. In a listening mode the assistant never speaks; in an answer-when-asked mode it creates a response only when the latest human utterance addresses it by name. Every outside participant hears a notice and is asked to consent before joining; each consenting leg is transcribed separately, mixed recording pauses while a non-consenting participant is present, and the conference ends when no human remains.

## Technical Field

Telephony conferencing, realtime speech-to-speech AI agents and turn-taking control.

## Background

Voice AI agents are built for one-to-one calls: after each pause in the caller's speech they take a turn. In a conference with several humans this makes the agent interrupt, answer remarks addressed to others and talk over people, so operators either keep it muted and prompt it manually or leave it out. Joining an agent to a conference also requires telling the speech bridge which agent and which conference a new media stream belongs to; passing that in a dialled number or a reusable parameter lets anyone who learns it attach a stream. Conferences also mix participants with different recording consent, but a single mixed recording cannot honour one person's refusal while recording everyone else, and per-speaker transcription is lost in a mixed track.

## Summary of the Disclosure

The conference service places the assistant as a participant whose call is answered by markup that connects a bidirectional media stream to a realtime bridge. The service mints a single-use join ticket bound to the conference and the assistant, with a short lifetime, carries it on the leg, and the bridge consumes it at the stream's start frame; an expired, unknown or reused ticket is refused. The bridge opens the assistant's realtime session with server voice activity detection but without automatic response creation. The organizer switches the assistant between listening only and answering when asked; in the latter, after each human utterance the bridge checks whether it addresses the assistant by name and only then requests a response, kept to a few short sentences. All legs hear that an assistant is listening. Outside participants hear a recording notice and are asked to consent before joining; each consenting human leg forks its inbound audio to a transcription worker; mixed recording stops while a non-consenting participant is present. The conference ends when no human participant remains.

## Detailed Description

Records. A conference record holds an identifier, the carrier's conference identifier, a name derived from the identifier, the organizer, status (created, active, ended, failed), timestamps and any recording identifier. A participant record holds the conference, the kind (application user, outside phone party, assistant), the user, phone number or assistant identifier, the call identifier, a label, mute state, when the notice was played, join and leave times, and status. Both are scoped to the organizer.

Joining humans. The organizer's leg starts the conference and its exit ends it, so no other legs are bridged without the organizer. Caps bound the number of human and assistant participants. Every leg carries a time limit. An outside phone guest is dialled with markup that first plays a recording notice and asks for consent by keypress or speech, records the answer and the time the notice played, and only then connects the guest; adding a phone guest is refused while no approved notice text is configured.

Joining the assistant. The service adds the assistant as a participant through the carrier's participant interface. The dialled leg is answered by markup that opens a bidirectional media stream to the realtime bridge. Before dialling, the service mints a single-use ticket bound to this conference and this assistant with a short lifetime, and carries it on the leg, for example in a SIP header or bound to the participant's call identifier. When the stream's start frame arrives, the bridge looks up and consumes the ticket; if it is unknown, expired or already used, the stream is closed. The assistant's ordinary access checks (that it is active and permitted to use voice) apply as on any call. On joining, every leg hears that an AI assistant has joined and is listening.

Turn control. The bridge opens the realtime model session with server-side voice activity detection enabled, so utterance boundaries and transcripts are still produced, but with automatic response creation disabled, so a pause never by itself triggers speech. The assistant has two modes set by the organizer. In listening-only mode the bridge never requests a response. In answer-when-asked mode, after each completed human utterance the bridge examines the latest utterance and requests a response only if it addresses the assistant by name; the assistant then answers in one to three short sentences. When allowed to speak, the assistant says its name once on joining so people know how to address it.

Transcription and recording. Each human leg that consented starts a fork of its inbound audio track to a transcription worker before it joins, so every speaker is transcribed separately and turns are attributed by participant; the assistant's words come from the bridge's transcript events. A mixed recording starts from the first consent and is stopped while any participant who declined is present. A participant who declines stays on the call but is not recorded, forked or transcribed. A rolling summary may be generated from the live transcript and streamed to the organizer's application.

Teardown. When the carrier reports that the last human left, or the organizer leaves, the conference and every remaining leg, including assistant legs, are ended. A conference also ends at a configured maximum duration after a spoken warning. A configuration switch disables the whole feature: routes answer as absent and a dial to a conference is refused with a spoken message.

## Disclosed Features (each disclosed alone and in combination)

1. A method adds an AI assistant to a phone conference as its own leg whose audio is streamed bidirectionally to a realtime speech bridge.
2. The method identifies the assistant and conference to the bridge by a single-use, short-lived ticket bound to both and consumed at the stream's start frame, refusing unknown, expired or reused tickets.
3. The method runs the assistant's realtime session with server voice activity detection and automatic response creation disabled.
4. The method offers a listening-only mode in which the assistant never speaks and an answer-when-asked mode in which a response is requested only when the latest human utterance addresses the assistant by name.
5. The method announces to every leg that an AI assistant is listening.
6. The method plays a recording notice and asks consent of each outside participant before connecting them, and records the answer.
7. The method forks each consenting human leg's inbound audio to transcription for per-speaker attribution.
8. The method stops the mixed recording while a non-consenting participant is present.
9. The method ends the conference and assistant legs when no human participant remains or the organizer leaves.

## Variations

- Name addressing may be detected by keyword spotting on the transcript, by a small classifier, or by the realtime model asked to output only a yes or no decision.
- The ticket may be a signed token verified by the bridge rather than a stored single-use row.
- The answer-when-asked mode may also respond to a push-to-talk signal from the organizer's application.
- Instead of stopping the mixed recording, the non-consenting participant's audio may be excluded from a per-leg recording mix.
- The assistant may address the organizer privately through the application instead of speaking into the conference.

## Note on omitted parameters

Ticket lifetime, participant caps, maximum duration and summary frequency are implementation choices and are not stated.
