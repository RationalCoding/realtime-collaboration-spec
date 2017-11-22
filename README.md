# Realtime Collaboration for Programming
## Working Draft

### Editors:
Thomas Mullen

## Abstract
This document defines a set of APIs for implementing code editor extensions that enable realtime collaboration between programmers (also known as pair-programming). The document is intended to standardize future implentations of these software and ensure implementations are cross-compatible, performant and feature-rich.

This specification is **not** feature complete and is expected to be change significantly. Members of the community are highly encouraged to contribute to this document.

A future test suite will be used to build an implemen  tation report of the API.

## Table of contents
1. [Introduction](#introduction) 
2. [Conformance](#conformance) 
3. [Terminology](#terminology) 
4. [CRDT Model](#crdt-model)
5. [Network Protocol](#network-protocol)
6. [Overlay Protocol](#overlay-protocol)
7. [Integration Recommendations](#integration-suggestions)

## Introduction
There are a number of facets to programmer-oriented realtime collaboration that are covered by this specification:
- Storing collaborative data in a way that is eventually consistent (by use of CRDTs).
- Connecting and sending data through widely-available network transports.
- Ensuring sent messages preserve casuality (a requirement of CRDTs).
- Allowing extensibility of the protocol to permit arbitrary data to be replicated.

This document defines the APIs used for these features.

## Conformance

## Terminology
The term **CRDT** is an acronym for Conflict-Free Replicated Data Type.

## CRDT Model
### Interfaces
This section specifies public interfaces that must be available to the extension developer.

### CRDTFileIndex
A `CRDTFileIndex` instance represents a map of path names to files.

```erlang
interface CRDTFileIndex {
  CRDTFile            createFile(String path);
  void                deleteFile(String path);
  void                moveFile(String path, String newPath);
  CRDTFile            getFile(String path);
  sequence<CRDTFile>  getFiles();
  
  attribute EventHandler    onFileCreate;
  attribute EventHandler    onFileDelete;
  attribute EventHandler    onFileMove;
}
```

### CRDTFile
A `CRDTFile` instance represents a sequence of characters and a sequence of cursors.

```erlang
interface CRDTFile {
  String                    getContent();
  String                    getCharAt(index);
  sequence<CursorPosition>  getCursors();
  void                      setCursors(sequence<CursorPosition>);
  void                      insert(string, position);
  void                      delete(position, length);
  void                      replaceRange(string, position, length);
  
  attribute EventHandler    onChange;
  attribute EventHandler    onCursorMove;
  attribute EventHandler    onFileDelete;
}
```

```erlang
object CursorPosition {
  unsigned long    line;
  unsigned long    char;
}
```

## Events

### FileChangeEvent
```erlang
object FileChangeEvent {
  String                         fileIdentifier;
  sequence<FileChangeInsertAtom> inserts;
  sequence<FileChangeDeleteAtom> deletes;
}
```

```erlang
object FileChangeInsertAtom {
  unsigned long   index;
  String          element;
}
```

```erlang
object FileChangeDeleteAtom {
  unsigned long   index;
}
```

### CursorMoveEvent

```erlang
object CursorMoveEvent {
  String          fileIdentifier;
  CursorPosition  position
}
```

*The following are working notes and not part of the specification*
- Sequences nested in a map CRDT, mapping file paths to sequences.
- Map could be implemented using Observe-Remove-Sets, sequences with LSEQ, Logoot, Woot, etc (likely with the Split optimization).
- Sequences support undo.
- Must allow diff-based merging for offline work (consensus protocol?).

## Network Protocol
*The following are working notes and not part of the specification*
- Must be available within web-based editors and more traditional IDEs.
- Ideally is peer-to-peer with some fallback (WebRTC and WebSockets fallback?)
- Must allow offline work.
- Must preserve the order messages are sent in (at least between two peers).
- Must guarantee delivery.

## Overlay Protocol

*The following are working notes and not part of the specification*
- Must preserve causality (as CRDTs assume this)
- Must guarantee eventual connectivity (the graph of peers is connected)
- Must be robust (graph must be able to reconnect)

## Integration Recommendations
*The following are working notes and not part of the specification*
- Non-normative suggestions for how to implement editor extensions
- "View extension" vs "Sync extension". View extensions only modify files within one "master" peer's workspace and don't persist code on the other peers' filesystems (eg Teletype, VSCode Live Share). Sync extensions modify files on all peers, and no peer has a unique role (eg Multihack, server-based solutions like CoVim). Either one should be possible to implement, as this specification will not assume anything about the filesystem. 

## Privacy And Security Considerations

## Acknowledgements
Formatting of this document is heavily influenced by W3 Candidate Recommendation documents, especially [WebRTC 1.0: Real-time Communication Between Browsers](https://www.w3.org/TR/webrtc).

## References

