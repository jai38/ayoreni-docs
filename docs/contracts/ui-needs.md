# What the UI needs from the backend

**Owner: the UI session.** Add one line per need. Also open an issue in `jai38/myhealthapp-backend` labelled `from-ui`, and put its number here. The backend session marks the line done, with its PR.

| Date | Screen (link to the design canvas) | Need | Issue | Status |
|---|---|---|---|---|
| 2026-11-19 | [S07](https://claude.ai/artifact/7f6T8pLUF6V4CTzCKeYmU9), [S06](https://claude.ai/artifact/7f6T8pLUF6V4CTzCKeYmU9) | Accept `MISSED` on the POST and PATCH dose action routes (only from the user's explicit choice); Undo to `PENDING` from it | [#5](https://github.com/jai38/myhealthapp-backend/issues/5) | Open |
| 2026-11-19 | [S07](https://claude.ai/artifact/7f6T8pLUF6V4CTzCKeYmU9) | Refuse a future `actionTime` on the POST dose action | [#6](https://github.com/jai38/myhealthapp-backend/issues/6) | Open |
| 2026-11-19 | [S07](https://claude.ai/artifact/7f6T8pLUF6V4CTzCKeYmU9) | Structured optional skip reason (Felt unwell, Doctor advised, Ran out, Side effects, Other) | [#7](https://github.com/jai38/myhealthapp-backend/issues/7) | Open |
| 2026-11-19 | [S06](https://claude.ai/artifact/7f6T8pLUF6V4CTzCKeYmU9) | `fullName` on `GET /auth/me`, for the greeting | [#8](https://github.com/jai38/myhealthapp-backend/issues/8) | Open |
