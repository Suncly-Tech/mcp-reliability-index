# MCP Reliability Index: method (pre-registered 23 November 2026)

## Question
Do the tools of popular remote MCP servers do what their own descriptions say?

## Releases
- Release 1 (November 2026; its publication date is recorded in the changelog): a ranking on what each server's public tool list declares (M3, M4 and M5 below). We send only the MCP handshake (`initialize` and the `initialized` notification) and `tools/list` (all pages); we call no tools, so no judge model is used. It reports what servers publicly declare, not whether their tools work.
- Release 2 (planned for mid-December 2026): the behaviour check below (M1 to M5).

Our client asks for protocol version 2025-06-18 and accepts the version the server agrees to. For every server we record its name and version as given in its `initialize` answer, the protocol version agreed, and the date and time (UTC) of the reading.

## Sample
- Release 1: 100 remote MCP servers from the best-known companies, by the web popularity of the company's domain, that answer without sign-in. Candidates are remote servers (Streamable HTTP or the older HTTP+SSE transport) listed in the official MCP Registry under a verified domain namespace (a name such as com.example/server, which the Registry accepts only from the owner of example.com), taking the latest listed version of each. Each candidate is ranked by its namespace's registrable domain (the domain its owner registered, under the Public Suffix List including its private section, so a server on shared hosting such as someone.workers.dev does not take its platform's rank; the list version is recorded in summary.json) in the Tranco list of popular websites, list 46YLX (tranco-list.eu, top 1 million, downloaded 23 November 2026; the file's SHA-256 is recorded in summary.json), best-known first; servers of one domain are taken in order of Registry name, and once one server of a domain is included, that domain's other servers are skipped. Servers are read in that order until 100 are included or the list ends; every server reached on the way is reported, either with its measures or as excluded with the reason. Servers whose domain has no Tranco rank, and servers published under a GitHub-account namespace (io.github.…), are not in Release 1; servers that need an account are not in Release 1. The Registry snapshot's date and time are recorded. A server is excluded if its listed URL needs account-specific values or a sign-in header, if it asks for a sign-in, API key or token (HTTP 401, HTTP 403 naming an authentication scheme, or an error message asking for one), does not answer within 30 seconds after one retry, answers with an error or with something that is not a valid MCP answer, returns an empty tool list, or gives two different tool lists in two readings taken the same day.
- Release 2: 100-200 remote MCP servers in three tiers: public servers needing no sign-in, official vendor servers tested with our own free or sandbox accounts, and the most-used community servers from the official MCP Registry. Servers whose terms forbid automated access are left out, and a server needs at least one read-only tool.
- For both: remote servers only; one entry per server even if it is listed in several places; servers where Suncly has a conflict of interest (for example its own customers) are left out unless the server's owner agrees in writing.

Every exclusion is listed with its reason in servers.csv, except servers left out for a conflict of interest, which are counted in summary.json but not named, to keep client relationships confidential.

## Scope and limits
- Read-only tools only. We never call a tool that creates, changes, sends or deletes. A tool counts as read-only only if its readOnlyHint annotation is true and a manual check of its name and description agrees; anything unclear is excluded and reported as not tested.
- Our own accounts only. No customer data, no borrowed tokens, never a payment card.
- One request at a time, at least 1 second apart. No load, rate-limit, fuzzing or security testing.
- We test behaviour against the tool's own description, for one server version on one date. We do not rate safety or security.
- If a server's owner asks us to stop, we stop at once, exclude the server and record the request.
- We publish no access tokens, API keys, account IDs or personal data.

## Procedure (Release 2)
1. Draft valid test calls for each read-only tool from its description and schema. Where a tool needs an ID, we first get a real one from another read-only tool on the same server; we never invent IDs.
2. Run each call 3 times in fresh sessions.
3. A judge model votes 3 times on whether each answer matches the description; a run's verdict is the majority of its 3 votes. A run that times out, hits a sign-in error or gets no answer is inconclusive. The judge model and its prompt are recorded in the changelog before the full run.
4. Per tool: Pass = at least 2 of 3 runs match. Fail = at least 2 of 3 runs contradict it AND a person confirms; if the person does not confirm, the person sets the verdict to pass or inconclusive and it is logged as overturned. Every other outcome is inconclusive; inconclusive is reported, never counted as fail.
5. A second person re-checks a random 10% of verdicts. Disagreements are logged and settled together.

## Measures (all reported for every server)
- M1 claim match (headline candidate, Release 2): passes / (passes + fails) over the read-only tools tested. Inconclusive tools are reported separately, never hidden. A server with no pass and no fail has no M1 value.
- M2 consistency (Release 2): share of tools with 3 answered runs whose answers differ in substance across the 3 identical runs, judged and confirmed the same way as M1. Differences in key order, whitespace, or values the description says change over time (such as timestamps or live data) do not count.
- M3 context cost: tokens in the server's tool list: the tools from all pages of tools/list combined into one array in the order returned, with every field as returned, serialised as compact JSON (no spaces, unicode not escaped), counted with tiktoken o200k_base. The count compares servers on one fixed scale; other models' tokenizers give somewhat different numbers.
- M4 safety labels: share of tools whose annotations explicitly include readOnlyHint or destructiveHint (with either value). Under the protocol's defaults, a tool without readOnlyHint is treated as one that may change data, and a tool that is not read-only and has no destructiveHint is treated as one whose changes may be destructive. Annotations were added in protocol version 2025-03-26, so each server's protocol version is reported next to its M4.
- M5 undocumented inputs, two numbers: the share of tools with no description, and the share of input parameters (top-level properties of inputSchema) with no description. A missing, empty or blank description counts as none; a title is not a description. A server with no input parameters has no value for the second number.

Each measure is reported per server. Across servers we report the median, the lowest and the highest value of each measure, and for M4 and M5 also the share over all tools (or all parameters) combined. Servers are ranked separately on each measure and on each of M5's two numbers, with no combined score: fewer tokens rank higher on M3, a higher share ranks higher on M4, and a lower share ranks higher on M5. Servers with equal values share a rank; a server with no value for a measure is listed unranked for it.

The raw initialize and tools/list answers are published in results/ next to the computed measures, so anyone can recompute every number.

The published headline is the most newsworthy of the measures in that release; all measures of a release are always reported for every server.

## Failure types (Release 2)
F1 wrong result, F2 ignored input, F3 valid input rejected, F4 output differs from declared schema, F5 silent failure.

## Disclosure
- Release 1 reports only what each server publicly declares in its tool list, so it is published without a warning period. Any builder may ask for a re-read after changing their tool list.
- Release 2: every builder gets their results privately at least 10 days before publication, with a free retest and a right to reply printed next to their result.

## Changelog
- 2026-11-23: method published.
- 2026-11-23, before any data was collected: the Registry holds about 40,000 qualifying remote servers, more than one release can read, so Release 1 is limited to 100 servers from the best-known companies (ranked by the Tranco rank of their verified domain, one server per domain); conflict-of-interest exclusions are counted, not named.
