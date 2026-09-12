
# PLAN.md
## Music Web MVP private kiểu Spotify — bản chốt triển khai, local-first, migration-only, skill-gated

> PRECONDITION: Codex phải đọc `D:\agent\README.md` → `D:\agent\MANIFEST.md` → `D:\music\AGENTS.md` → plan này → `web\PHASES.md` → relevant skills trước khi implementation.

Ngày chốt: 12/09/2026  
Chủ dự án: vinh17491  
Repo duy nhất: `D:\music`  
Ứng dụng web: `D:\music\web`  
Mục tiêu: WEB MVP responsive cho khoảng 10–20 người, invite-only, phi thương mại, phục vụ học tập/đồ án và sử dụng riêng. Bản này KHÔNG bao gồm app native iOS/Android, PWA installable, offline download hoặc phát nền native. Nếu các đầu ra đó trở thành bắt buộc thì mở một khối P7/P8 mới thay vì nhét vào MVP hiện tại.

---

# 0. TUYÊN BỐ CHẤT LƯỢNG VÀ NGUYÊN TẮC 100/100

File này không hứa “code không bao giờ có lỗi”. “100/100” trong tài liệu này có nghĩa là kế hoạch đã khóa được scope, kiến trúc, nguồn sự thật, security boundary, test gate, rollback, audit và tiêu chí nghiệm thu đến mức Codex không được tự suy đoán các quyết định kiến trúc quan trọng. Điểm 100/100 cuối cùng chỉ được xác nhận khi toàn bộ gate bắt buộc có evidence thực tế và không còn phase ở trạng thái TODO hoặc FAIL.

Codex phải đọc `D:\agent\README.md` và project `AGENTS.md` trước file này. File này là source-of-truth cho thứ tự implementation, architecture và acceptance criteria của dự án. Nếu code hiện hữu, README, comment, file cũ hoặc lời suy đoán của agent mâu thuẫn với file này thì phải dừng phase hiện tại, báo cáo mâu thuẫn và chỉ sửa theo hướng đưa project trở về đúng plan. Không được “tối ưu thêm”, đổi framework, đổi database, đổi provider hoặc thêm thư viện chỉ vì agent thấy tiện hơn.

Mỗi phase chỉ được làm đúng phạm vi phase đó. Sau khi thực hiện xong, phase bắt buộc phải tự audit bằng các lệnh, test và kiểm tra file được nêu. Nếu audit fail, phase chưa hoàn thành. Codex phải ghi lỗi, nguyên nhân gốc, file bị ảnh hưởng, cách sửa, test hồi quy và chỉ được đánh PASS sau khi chạy lại audit xanh. Không được đẩy lỗi sang phase sau.

Trạng thái phase chỉ có bốn giá trị: `TODO`, `PASS`, `FAIL`, `SKIPPED-NOT-NEEDED`. `SKIPPED-NOT-NEEDED` chỉ dùng cho phase dự phòng có điều kiện và phải có lý do. Gate cuối yêu cầu `0 TODO`, `0 FAIL`; các phase dự phòng không phát sinh có thể `SKIPPED-NOT-NEEDED`.

Mọi phase làm thay đổi code hoặc migration phải có commit riêng, trừ phase chỉ quan sát hoặc test. Commit theo mẫu `<type>(<phase>): <mo ta khong dau>`. Không force-push. Không commit secret. Không sửa history migration đã được push production; thay đổi mới phải là migration mới.

---

# 0A. CONTINUOUS EXECUTION CONTRACT — KHÔNG ĐƯỢC DỪNG Ở PREFLIGHT

B-1 và B-2 chỉ là bootstrap/precondition. Chúng KHÔNG phải deliverable cuối và không được dùng để tuyên bố task hoàn thành.

TASK của Codex là thực hiện **toàn bộ plan này**, bắt đầu từ phase đầu tiên chưa có trạng thái PASS/SKIPPED hợp lệ và tiếp tục tuần tự cho đến Z7.8.

Sau mỗi phase PASS, Codex phải tự:
1. cập nhật evidence trong `web/PHASES.md`;
2. review diff;
3. commit/push recovery state theo Git policy;
4. xác định phase kế tiếp;
5. load thêm skill nếu phase kế tiếp cần;
6. bắt đầu phase kế tiếp mà KHÔNG yêu cầu người dùng xác nhận lại.

Một phase FAIL do code/test/runtime KHÔNG phải lý do dừng task. Codex phải dùng `systematic-debugging`, tìm root cause, sửa, chạy regression/audit lại và tiếp tục.

Codex chỉ được dừng trước Z7.8 khi tồn tại **hard external blocker** mà nó thật sự không thể tự giải quyết bằng tool/quyền hiện có, ví dụ:
- cần người dùng hoàn tất browser login/2FA/OAuth;
- cần Windows elevation/reboot hoặc mở Docker Desktop thủ công mà tool không làm được;
- thiếu secret/credential mà project chưa có và không thể tự sinh;
- có nguy cơ ghi/xóa dữ liệu ngoài scope và instruction không cho phép tự quyết;
- provider/platform bên ngoài outage kéo dài làm acceptance criterion không thể kiểm chứng.

Khi gặp hard blocker, Codex không được nói “project complete”. Nó phải:
- đánh phase hiện tại FAIL/BLOCKED rõ ràng;
- ghi exact blocker + command/output evidence;
- commit/push checkpoint an toàn nếu có thể;
- ghi `RESUME FROM: <phase>`;
- nói chính xác hành động duy nhất người dùng cần thực hiện.

Nếu session/context/runtime buộc phải kết thúc nhưng không có hard blocker, Codex phải tạo recovery checkpoint, cập nhật PHASES và báo `SESSION CHECKPOINT — PROJECT NOT COMPLETE`, không được dùng từ “done/complete” cho toàn dự án.

Chỉ Z7.8 PASS mới cho phép tuyên bố toàn dự án hoàn thành.

# 0B. CANONICAL PLAN FILE — CHỈ MỘT NGUỒN KẾ HOẠCH

Canonical plan path cố định của dự án là:

`D:\music\PLAN.md`

Mọi file plan cũ như `PLAN-FINAL-*.md`, `plan-full-class*.md`, bản 105/160/168/172/176 trước đó đều là superseded historical artifacts và KHÔNG được dùng để điều khiển implementation.

Codex không được merge yêu cầu từ nhiều bản plan cũ. Nếu các file cũ vẫn nằm trong repo, chỉ `PLAN.md` có authority cho phase order/acceptance criteria. Có thể chuyển file cũ vào `archive/plans/` bằng một commit housekeeping có audit, nhưng không được xóa lịch sử Git để “làm sạch”.

Nếu `AGENTS.md`, README hoặc prompt cũ trỏ tới tên plan khác, Codex phải cập nhật project-level reference sang `D:\music\PLAN.md` trước implementation. Không sửa `D:\agent` để giải quyết mismatch.

# 1. KIẾN TRÚC ĐƯỢC KHÓA

GitHub repository visibility được khóa là PUBLIC tại `https://github.com/vinh17491/music`. Đây chỉ là visibility của source code; ứng dụng vẫn invite-only/private và mọi secret/user data không được commit.

Database là PostgreSQL do Supabase cung cấp. Môi trường local dùng Supabase Local để có cùng PostgreSQL, Auth, Storage, REST và RLS behavior cần thiết cho ứng dụng. Dự án dùng workflow **migration-only**. Không tạo `supabase/schemas/`. Không dùng `docs/sql` làm nguồn sự thật. Nguồn sự thật database duy nhất là `web/supabase/migrations/*.sql`.

Cấu trúc database mong muốn được tái tạo hoàn toàn bằng `npx supabase db reset`. Nếu một database local chỉ chạy được nhờ SQL đã gõ tay trong Studio nhưng `db reset` không dựng lại được thì phase database đó FAIL.

Luồng bắt buộc là: viết migration mới → `npx supabase db reset` → database tests → app/integration tests → commit migration → trước deploy chạy `npx supabase db push --dry-run` → chỉ khi output đúng mới `npx supabase db push` lên project cloud đã link.

Seed chỉ dành cho local/test. Không chạy `--include-seed` lên production. Không dùng production data làm seed.

Các PostgreSQL schema namespace được phép dùng là `public` cho bảng ứng dụng, `private` cho helper security-definer không muốn expose Data API, và schema hệ thống `auth`/`storage` của Supabase. “Bỏ schema” trong project chỉ có nghĩa bỏ thư mục declarative `supabase/schemas/`; không được phá các PostgreSQL namespace nói trên.

Frontend/backend là Next.js 16 App Router, TypeScript strict, Tailwind, TanStack Query cho server state, Zustand cho player state và Howler cho playback. Supabase Auth quản lý đăng nhập. Supabase Storage dùng bucket private. Redis/Upstash chỉ dùng cho rate-limit và trạng thái provider nếu thật sự cần. Không thêm ORM như Prisma trong MVP này.

Nguồn nhạc ngoài là Jamendo và Audius. Jamendo credential chỉ dùng server-side. Audius phải dùng API credential hiện hành; không được giả định discovery provider anonymous “không cần key”. Bearer token Audius tuyệt đối không được đưa vào client bundle.

---

# 2. CẤU TRÚC THƯ MỤC CHUẨN

```text
D:\music\
├─ .git\
├─ .github\
│  └─ workflows\
├─ AGENTS.md
├─ PLAN.md
├─ README.md
└─ web\
   ├─ src\
   │  ├─ app\
   │  ├─ components\
   │  ├─ hooks\
   │  ├─ lib\
   │  ├─ stores\
   │  └─ types\
   ├─ e2e\
   ├─ supabase\
   │  ├─ config.toml
   │  ├─ migrations\
   │  ├─ tests\
   │  │  └─ database\
   │  └─ seed.sql
   ├─ .env.example
   ├─ package.json
   ├─ package-lock.json
   ├─ playwright.config.ts
   └─ next.config.ts
```

Nếu Codex phát hiện cùng một chức năng được implement ở hai đường dẫn khác nhau, ví dụ vừa `src/lib/player-store.ts` vừa `src/stores/player-store.ts`, phải dừng và hợp nhất về vị trí chuẩn của plan trước khi tiếp tục.

---

# 3. PROTOCOL AUDIT BẮT BUỘC SAU MỖI PHASE

Sau mỗi phase, Codex phải tạo một báo cáo ngắn trong `web/PHASES.md`. Báo cáo phải ghi cả `SKILLS LOADED` để chứng minh phase đã dùng đúng procedural guidance từ Agent Library. Báo cáo phải ghi mã phase, trạng thái, file đã thay đổi, lệnh audit đã chạy, kết quả thực tế, lỗi gặp phải nếu có và commit/tag tương ứng. Không được chỉ ghi “done”.

Nếu audit phát hiện lỗi, Codex phải ưu tiên sửa nguyên nhân gốc thay vì che symptom. Không được thêm `any`, `@ts-ignore`, `eslint-disable`, bỏ RLS, đổi bucket thành public, tắt test hoặc catch lỗi rồi trả dữ liệu giả chỉ để gate xanh. Nếu bắt buộc có ngoại lệ, phải ghi accepted risk vào `DECISIONS.md` cùng lý do và ngưỡng cần nâng cấp.

Sau phase có migration, audit tối thiểu luôn gồm `npx supabase db reset` và test liên quan. Sau phase thay TypeScript, audit tối thiểu gồm `npx tsc --noEmit` và `npm run lint`. Sau phase thay route hoặc runtime, phải có smoke test thật. Sau phase thay player, phải có playback test thật. Sau phase security, phải có negative test chứng minh request không hợp lệ bị từ chối.

---

# 4. ENVIRONMENT CONTRACT

`.env.local` không được commit. `.env.example` chỉ chứa tên biến và chú thích, không chứa secret thật. Bộ biến mục tiêu gồm Supabase URL/anon key cho môi trường chạy, service role server-only, Jamendo client ID, Audius API key, Audius bearer token nếu backend flow cần, Upstash URL/token nếu rate-limit production bật, và `CRON_SECRET`. Biến có thể thay đổi theo SDK hiện hành nhưng mọi thay đổi phải được ghi vào plan/README trước khi code dựa vào nó.

Service role và Audius bearer token là server-only. Bất kỳ import nào khiến module chứa các secret này đi vào `use client` đều là lỗi blocking.

---

# 5. CÁC PHASE TRIỂN KHAI


# KHỐI B-1 — AGENT BOOTSTRAP + SKILL GATE

Khối này chạy trước G0 trong mọi dự án mới và được lặp lại ở mức session-resume khi Codex mất context. Mục tiêu là bảo đảm Codex hiểu cách làm việc trước khi chạm vào code. `D:\agent\` chỉ được đọc; project mới là nơi được sửa.

## B-1.1 — Đọc Universal Agent Bootstrap

**Thực hiện.** Codex đọc toàn bộ `D:\agent\README.md`. Không skim phần bootstrap, protected library, skill loading, debugging, testing và verification. Chưa được implement project.

**Audit bắt buộc trước khi rời phase.** Codex phải báo lại ngắn gọn năm invariant: `D:\agent\` read-only; inspect-before-change; progressive skill loading; evidence-before-completion; project instructions/plan điều khiển architecture. Nếu không thể đọc file hoặc file thiếu thì FAIL và không code.

**Khi audit FAIL.** Không tự bịa nội dung README. Báo path/file issue và chờ file được khôi phục hoặc chỉ tiếp tục khi có instruction cao hơn cho phép.

## B-1.2 — Đọc MANIFEST và lập skill inventory cho project

**Thực hiện.** Đọc `D:\agent\MANIFEST.md` để xác định skill hiện có. Không đọc toàn bộ 626 skill. Ghi nhóm skill có khả năng dùng cho project: executing plans, worktrees, TDD, debugging, verification, Supabase, Supabase Postgres, migrations, Next.js, TypeScript, Zod, TanStack Query, Vitest, Playwright, security, Vercel, accessibility và code review.

**Audit bắt buộc trước khi rời phase.** Mỗi skill được ghi bằng logical name và trigger sử dụng. Nếu MANIFEST có duplicate entry, không xem duplicate là hai requirement; dùng discovery theo tên/path thật.

**Khi audit FAIL.** Nếu skill được plan yêu cầu nhưng MANIFEST không có, báo missing capability; không tự tạo skill giả.

## B-1.3 — Đọc project AGENTS.md

**Thực hiện.** Đọc toàn bộ `D:\music\AGENTS.md`. File này khóa quy tắc local project: migration-only, anti-task-jump, skill routing, security invariants, debug protocol, completion protocol và phase report.

**Audit bắt buộc trước khi rời phase.** Xác nhận `AGENTS.md` không yêu cầu sửa `D:\agent`. Xác nhận plan canonical filename đúng. Nếu AGENTS và plan mâu thuẫn, dừng và ghi conflict trước code.

**Khi audit FAIL.** Sửa project instruction chỉ khi user/plan cho phép; tuyệt đối không “giải quyết” bằng sửa Agent Library.

## B-1.4 — Đọc plan canonical toàn bộ

**Thực hiện.** Đọc `PLAN.md` từ đầu đến cuối ít nhất một lần trước implementation đầu tiên. Ở session sau có thể đọc lại phần global rules + block hiện tại + dependencies, nhưng khi context nghi ngờ phải đọc lại toàn bộ.

**Audit bắt buộc trước khi rời phase.** Codex xác định scope, architecture, database source-of-truth, phase hiện tại, prerequisite và acceptance evidence. Chưa được sửa code.

**Khi audit FAIL.** Nếu phase/dependency không rõ hoặc plan tự mâu thuẫn, lập plan-deviation report thay vì tự suy đoán.

## B-1.5 — Inspect repo và isolation

**Thực hiện.** Inspect git root, current branch/worktree, dirty files và user changes. Áp dụng skill `using-git-worktrees`: detect isolation trước; không tự tạo nested worktree. Nếu cần tạo worktree và chưa có user preference/permission, xử lý theo skill/harness hiện hành.

**Audit bắt buộc trước khi rời phase.** Báo repo root, branch/worktree state, dirty files và baseline. Không được overwrite unrelated user work.

**Khi audit FAIL.** Dirty state không hiểu nguồn gốc hoặc merge conflict là blocker; không reset/clean cưỡng bức.

## B-1.6 — Chọn skill theo current phase

**Thực hiện.** Dựa vào AGENTS skill-routing matrix và phase hiện tại, chọn minimum relevant skills. Đọc mỗi SKILL.md đầy đủ. Chỉ đọc references/rules/scripts mà skill yêu cầu hoặc task thật sự cần. Supabase task luôn load Supabase skill; DB task thêm Supabase Postgres + migration skill; bug thêm systematic-debugging; trước PASS dùng verification-before-completion.

**Audit bắt buộc trước khi rời phase.** Phase report ghi `SKILLS LOADED` và lý do cho từng skill. Skill không liên quan không được load chỉ để “cho chắc”.

**Khi audit FAIL.** Nếu Codex phát hiện đã code trước khi đọc skill bắt buộc, phase implementation đó chưa được coi là audited; phải đọc skill, review lại code theo skill và chạy lại verification.

## B-1.7 — Lập phase execution contract

**Thực hiện.** Trước code, Codex mô tả outcome của phase, file dự kiến chạm, invariant phải giữ, test/audit command cần PASS và rollback/fix boundary. Search code hiện tại để tránh duplicate abstraction.

**Audit bắt buộc trước khi rời phase.** Contract phải đủ cụ thể để sau implementation có thể chứng minh PASS/FAIL mà không đổi tiêu chí giữa chừng.

**Khi audit FAIL.** Không code khi chưa biết evidence nào chứng minh phase xong.

## B-1.8 — Bootstrap gate

**Thực hiện.** Xác nhận B-1.1 tới B-1.7 đều PASS. Từ đây mới được bắt đầu G0 hoặc current implementation phase.

**Audit bắt buộc trước khi rời phase.** Codex xuất preflight report: repo/worktree, current phase, prerequisite, selected skills, expected files, audit evidence và known risks. `D:\agent\` phải có zero modifications.

**Khi audit FAIL.** Không triển khai project. Fix bootstrap/instruction conflict trước.




# KHỐI B-2 — TASK CONTINUATION + STOP-CONDITION GATE

## B-2.1 — Xác nhận TASK thật sự là toàn bộ project

**Thực hiện.** Codex đọc phần `TASK:` của startup prompt và xác nhận deliverable không phải preflight, không phải một phase đơn lẻ, mà là triển khai toàn bộ plan từ phase đầu tiên chưa PASS tới Z7.8. Nếu prompt chỉ có bootstrap mà không có TASK, Codex phải coi đó là instruction thiếu và không được tự tuyên bố project đã xong.

**Audit bắt buộc trước khi rời phase.** Preflight report phải ghi nguyên văn semantic: `PROJECT TASK ACTIVE: execute all remaining phases through Z7.8`. Phải xác định phase đầu tiên chưa PASS.

**Khi audit FAIL.** Không code và không tuyên bố completion. Báo thiếu TASK hoặc task conflict.

## B-2.2 — Khóa phase loop tự động

**Thực hiện.** Thiết lập vòng thực thi: `select first incomplete phase → load relevant skills → implement → audit → debug nếu fail → verify → update PHASES → commit/push → next phase`. Không hỏi “có muốn tiếp tục không?” giữa các phase khi requirement đã rõ.

**Audit bắt buộc trước khi rời phase.** Report phải nêu phase hiện tại và phase kế tiếp dự kiến nếu PASS. B-1.8 không được xuất hiện như endpoint.

**Khi audit FAIL.** Sửa execution state trước khi implementation.

## B-2.3 — Khóa stop conditions

**Thực hiện.** Phân biệt lỗi kỹ thuật với hard external blocker. Build fail, test fail, lint fail, migration fail, runtime bug, RLS bug hoặc provider adapter bug là việc phải debug; không phải lý do tự kết thúc Goal. Chỉ các blocker ngoài khả năng tool/quyền hiện tại mới cho phép dừng.

**Audit bắt buộc trước khi rời phase.** Codex liệt kê stop conditions đang áp dụng và xác nhận `normal implementation failure => debug and continue`.

**Khi audit FAIL.** Không thực thi vì agent có nguy cơ dừng sớm hoặc bỏ qua lỗi.

## B-2.4 — Resume/checkpoint contract

**Thực hiện.** Nếu session bị cắt, Codex phải để project ở trạng thái có thể tiếp tục: PHASES cập nhật, branch/commit/push checkpoint nếu an toàn, working tree được giải thích, và `RESUME FROM` chỉ đúng phase đầu tiên chưa PASS. Session sau phải đọc lại README/AGENTS/plan/PHASES rồi tiếp tục phase đó.

**Audit bắt buộc trước khi rời phase.** Mô phỏng resume logic từ tracker. Không có phase nào được PASS chỉ vì code đã tồn tại; phải có evidence.

**Khi audit FAIL.** Không bắt đầu long-running implementation loop.

# KHỐI G0 — GOVERNANCE + FOUNDATION


## G0.1 — Khóa scope MVP và anti-scope-creep

**Thực hiện.** Codex đọc toàn bộ file này, tạo `DECISIONS.md` và ghi rõ web MVP private 10–20 người, invite-only, không native app, không PWA installable, không offline download, không public signup. Không được bắt đầu code trước khi các giới hạn này được ghi trong repo.

**Audit bắt buộc trước khi rời phase.** Audit bằng cách đối chiếu README, DECISIONS và plan. Nếu bất kỳ tài liệu nào nói public signup, native app hoặc scope khác thì sửa tài liệu trước. PASS khi chỉ còn một scope thống nhất.

**Khi audit FAIL.** Phase chưa hoàn thành. Codex phải ghi symptom, reproduction, expected, actual, root cause, file bị ảnh hưởng, cách sửa, regression test và residual risk vào báo cáo lỗi. Không được sang phase sau, không được tắt test, nới RLS, chuyển bucket public, thêm `any`/`@ts-ignore` hoặc catch lỗi giả thành công. Sau khi sửa phải chạy lại toàn bộ audit của phase này rồi mới đổi trạng thái sang PASS.


## G0.2 — Khóa source-of-truth database

**Thực hiện.** Codex ghi trong `DECISIONS.md` rằng database dùng Supabase Local + PostgreSQL + migration-only. Cấm tạo `supabase/schemas/`, cấm dùng `docs/sql` làm nguồn thật và cấm thay bằng Prisma migrations.

**Audit bắt buộc trước khi rời phase.** Audit bằng `Get-ChildItem web\supabase -Recurse` sau khi cấu trúc tồn tại. Nếu thấy thư mục `schemas` hoặc SQL nguồn thật ngoài migrations thì phải hợp nhất hoặc xóa theo đúng migration history trước khi PASS.

**Khi audit FAIL.** Phase chưa hoàn thành. Codex phải ghi symptom, reproduction, expected, actual, root cause, file bị ảnh hưởng, cách sửa, regression test và residual risk vào báo cáo lỗi. Không được sang phase sau, không được tắt test, nới RLS, chuyển bucket public, thêm `any`/`@ts-ignore` hoặc catch lỗi giả thành công. Sau khi sửa phải chạy lại toàn bộ audit của phase này rồi mới đổi trạng thái sang PASS.


## G0.3 — Khóa quy tắc Git an toàn

**Thực hiện.** Khởi tạo hoặc xác nhận repo duy nhất ở `D:\music`. Không được có `.git` lồng trong `web`. Tạo quy ước branch/commit, cấm force push và cấm rewrite migration production.

**Audit bắt buộc trước khi rời phase.** Audit bằng `git rev-parse --show-toplevel`, `git status`, tìm `.git` lồng nhau. PASS khi root đúng `D:\music` và working tree có trạng thái hiểu được.

**Khi audit FAIL.** Phase chưa hoàn thành. Codex phải ghi symptom, reproduction, expected, actual, root cause, file bị ảnh hưởng, cách sửa, regression test và residual risk vào báo cáo lỗi. Không được sang phase sau, không được tắt test, nới RLS, chuyển bucket public, thêm `any`/`@ts-ignore` hoặc catch lỗi giả thành công. Sau khi sửa phải chạy lại toàn bộ audit của phase này rồi mới đổi trạng thái sang PASS.


## G0.4 — Tạo PHASES tracker chuẩn

**Thực hiện.** Tạo `web/PHASES.md` với tất cả phase của file này và trạng thái TODO ban đầu. Mỗi dòng phải có mã, trạng thái, ngày, commit/tag và evidence ngắn.

**Audit bắt buộc trước khi rời phase.** Audit đếm số phase tracker và số phase trong plan phải bằng nhau. Nếu lệch một mã thì FAIL vì Codex có thể bỏ sót công việc.

**Khi audit FAIL.** Phase chưa hoàn thành. Codex phải ghi symptom, reproduction, expected, actual, root cause, file bị ảnh hưởng, cách sửa, regression test và residual risk vào báo cáo lỗi. Không được sang phase sau, không được tắt test, nới RLS, chuyển bucket public, thêm `any`/`@ts-ignore` hoặc catch lỗi giả thành công. Sau khi sửa phải chạy lại toàn bộ audit của phase này rồi mới đổi trạng thái sang PASS.


## G0.5 — Tạo mẫu báo cáo lỗi

**Thực hiện.** Tạo `web/ERROR-REPORT-TEMPLATE.md` quy định khi fail phải ghi symptom, reproduction, expected, actual, root cause, files touched, fix, regression tests và residual risk. Không cho phép báo cáo kiểu 'đã sửa'.

**Audit bắt buộc trước khi rời phase.** Audit bằng cách đọc mẫu và mô phỏng một lỗi giả. PASS nếu người khác có thể tái hiện lỗi chỉ từ báo cáo.

**Khi audit FAIL.** Phase chưa hoàn thành. Codex phải ghi symptom, reproduction, expected, actual, root cause, file bị ảnh hưởng, cách sửa, regression test và residual risk vào báo cáo lỗi. Không được sang phase sau, không được tắt test, nới RLS, chuyển bucket public, thêm `any`/`@ts-ignore` hoặc catch lỗi giả thành công. Sau khi sửa phải chạy lại toàn bộ audit của phase này rồi mới đổi trạng thái sang PASS.


## G0.6 — Kiểm kê toolchain Windows và prerequisite local stack

**Thực hiện.** Xác nhận Node >=20, npm, Git và GitHub CLI hoạt động. Ghi version thật. Supabase CLI KHÔNG bắt buộc global; project sẽ pin CLI bằng npm dev dependency tại F1.1 và chạy bằng `npx supabase`. Kiểm tra Docker Desktop/container runtime; nếu chưa có thì ghi prerequisite cho F1.2 thay vì coi bootstrap/project hoàn tất.

**Audit bắt buộc trước khi rời phase.** Lưu output `node -v`, `npm -v`, `git --version`, `gh --version`. Với Docker chạy `docker version`; nếu command chưa tồn tại thì ghi `DOCKER PREREQUISITE PENDING F1.2`. Không yêu cầu `supabase --version` global.

**Khi audit FAIL.** Node/npm/Git/gh thiếu là blocker foundation. Docker thiếu được chuyển thành prerequisite bắt buộc của F1.2, không phải lý do kết thúc toàn Goal.

## G0.7 — Khóa Node và package manager

**Thực hiện.** Tạo `.nvmrc` hoặc trường `engines` phù hợp với Node 24.x đã chọn và dùng npm duy nhất. Không được sinh đồng thời yarn.lock/pnpm-lock.yaml.

**Audit bắt buộc trước khi rời phase.** Audit tìm lockfile. PASS khi chỉ có `package-lock.json` và CI dùng npm tương ứng.

**Khi audit FAIL.** Phase chưa hoàn thành. Codex phải ghi symptom, reproduction, expected, actual, root cause, file bị ảnh hưởng, cách sửa, regression test và residual risk vào báo cáo lỗi. Không được sang phase sau, không được tắt test, nới RLS, chuyển bucket public, thêm `any`/`@ts-ignore` hoặc catch lỗi giả thành công. Sau khi sửa phải chạy lại toàn bộ audit của phase này rồi mới đổi trạng thái sang PASS.


## G0.8 — Tạo Next.js app

**Thực hiện.** Tạo `web` bằng create-next-app Next 16, TypeScript, App Router, `--src-dir`, Tailwind, ESLint và import alias `@/*`. Không cài thư viện chức năng khác trong phase này.

**Audit bắt buộc trước khi rời phase.** Audit chạy `npm run dev`, `npx tsc --noEmit`, `npm run lint`, `npm run build`. PASS khi trang mặc định chạy, F12 không có runtime error và production build xanh.

**Khi audit FAIL.** Phase chưa hoàn thành. Codex phải ghi symptom, reproduction, expected, actual, root cause, file bị ảnh hưởng, cách sửa, regression test và residual risk vào báo cáo lỗi. Không được sang phase sau, không được tắt test, nới RLS, chuyển bucket public, thêm `any`/`@ts-ignore` hoặc catch lỗi giả thành công. Sau khi sửa phải chạy lại toàn bộ audit của phase này rồi mới đổi trạng thái sang PASS.


## G0.9 — Khóa đường dẫn src

**Thực hiện.** Kiểm tra mọi application code nằm trong `web/src`. Config, tests, migrations và docs có thể nằm ngoài `src` theo cấu trúc chuẩn. Sửa ngay mọi đường dẫn cũ gây nhầm.

**Audit bắt buộc trước khi rời phase.** Audit bằng tìm kiếm `app/`, `components/`, `stores/` ở sai root. PASS khi không tồn tại cấu trúc App Router thứ hai.

**Khi audit FAIL.** Phase chưa hoàn thành. Codex phải ghi symptom, reproduction, expected, actual, root cause, file bị ảnh hưởng, cách sửa, regression test và residual risk vào báo cáo lỗi. Không được sang phase sau, không được tắt test, nới RLS, chuyển bucket public, thêm `any`/`@ts-ignore` hoặc catch lỗi giả thành công. Sau khi sửa phải chạy lại toàn bộ audit của phase này rồi mới đổi trạng thái sang PASS.


## G0.10 — Tạo CI nền

**Thực hiện.** Tạo GitHub Actions chạy `npm ci`, TypeScript check, lint và build trong working-directory `web`. Chưa thêm test database trước khi Supabase local được cấu hình.

**Audit bắt buộc trước khi rời phase.** Audit chạy các lệnh CI local và sau push xác nhận job remote xanh. Nếu remote khác local phải sửa CI, không được bỏ step.

**Khi audit FAIL.** Phase chưa hoàn thành. Codex phải ghi symptom, reproduction, expected, actual, root cause, file bị ảnh hưởng, cách sửa, regression test và residual risk vào báo cáo lỗi. Không được sang phase sau, không được tắt test, nới RLS, chuyển bucket public, thêm `any`/`@ts-ignore` hoặc catch lỗi giả thành công. Sau khi sửa phải chạy lại toàn bộ audit của phase này rồi mới đổi trạng thái sang PASS.


## G0.11 — Khóa env separation

**Thực hiện.** Tạo `.env.example` với tên biến và comment server/client. Tạo `.env.local` chỉ trên máy. Xác nhận `.gitignore` bỏ qua `.env.local` và các file secret tương tự.

**Audit bắt buộc trước khi rời phase.** Audit bằng `git check-ignore -v web\.env.local` và `git status --short`. Nếu env local xuất hiện trong status thì phase FAIL.

**Khi audit FAIL.** Phase chưa hoàn thành. Codex phải ghi symptom, reproduction, expected, actual, root cause, file bị ảnh hưởng, cách sửa, regression test và residual risk vào báo cáo lỗi. Không được sang phase sau, không được tắt test, nới RLS, chuyển bucket public, thêm `any`/`@ts-ignore` hoặc catch lỗi giả thành công. Sau khi sửa phải chạy lại toàn bộ audit của phase này rồi mới đổi trạng thái sang PASS.


## G0.12 — Foundation checkpoint

**Thực hiện.** Chụp checkpoint bằng commit/tag nền sau khi G0.1–G0.11 đều PASS. Tag chỉ là mốc khôi phục, không phải bằng chứng app hoàn chỉnh.

**Audit bắt buộc trước khi rời phase.** Audit clone/checkout checkpoint trong thư mục tạm nếu thuận tiện và chạy build. PASS nếu foundation tái lập được từ Git sạch.

**Khi audit FAIL.** Phase chưa hoàn thành. Codex phải ghi symptom, reproduction, expected, actual, root cause, file bị ảnh hưởng, cách sửa, regression test và residual risk vào báo cáo lỗi. Không được sang phase sau, không được tắt test, nới RLS, chuyển bucket public, thêm `any`/`@ts-ignore` hoặc catch lỗi giả thành công. Sau khi sửa phải chạy lại toàn bộ audit của phase này rồi mới đổi trạng thái sang PASS.


# KHỐI F1 — SUPABASE LOCAL + MIGRATION-ONLY FOUNDATION


## G0.13 — Xác minh GitHub `vinh17491/music` + origin + quyền push

**Thực hiện.** Chạy `gh auth status --active --hostname github.com` và xác nhận active account là `vinh17491`. Nếu cần thì switch sang `vinh17491`; nếu chưa login thì dùng browser auth. Sau đó kiểm tra `gh repo view vinh17491/music`. Nếu repo chưa tồn tại và local chưa có origin, tự tạo PUBLIC repo bằng `gh repo create vinh17491/music --public --source=. --remote=origin --push`. Nếu repo đã tồn tại nhưng origin chưa có, thêm `https://github.com/vinh17491/music.git`. Nếu origin đã có nhưng URL khác target này, dừng `GIT REMOTE MISMATCH`, không tự sửa.

**Audit bắt buộc trước khi rời phase.** `gh auth status` xác nhận đúng account `vinh17491`; `git remote -v` cho thấy `origin` trỏ đúng `https://github.com/vinh17491/music.git` hoặc SSH-equivalent của cùng repo; `git fetch origin --prune` thành công; current local history đã có remote recovery pointer. Repo phải PUBLIC nếu Codex là bên tạo mới; app runtime vẫn invite-only/private.

**Khi audit FAIL.** Ghi `GIT BACKUP BLOCKER`; không bắt đầu migration/security/refactor lớn khi target GitHub hoặc authentication chưa được xác minh.


## G0.14 — Khóa branch-per-phase policy

**Thực hiện.** Từ phase này, mọi phase sửa file phải chạy trên branch `codex/<phase>-<slug>`. Branch tạo từ base đã xác định rõ và được push upstream ngay khi tạo. Nếu harness đã cung cấp worktree/branch isolation, tái sử dụng nó thay vì tạo nested worktree.

**Audit bắt buộc trước khi rời phase.** Báo current branch, upstream và `git status`. Current branch không được là `main`/`master` khi bắt đầu implementation.

**Khi audit FAIL.** Không sửa file cho tới khi branch isolation rõ ràng.

## G0.15 — Auto commit + push sau PASS

**Thực hiện.** Khi một phase PASS audit, Codex review diff, cập nhật PHASES, tạo commit đúng convention và `git push` branch hiện tại. Remote push lên `vinh17491/music` là một phần của evidence. Phase FAIL không được commit dưới nhãn PASS.

**Audit bắt buộc trước khi rời phase.** `git status` sạch hoặc chỉ còn thay đổi đã được ghi rõ là ngoài scope; `git log -1` đúng phase; `git status -sb` hiển thị branch đã đồng bộ với upstream.

**Khi audit FAIL.** Không sang phase kế tiếp cho tới khi trạng thái Git và remote backup được hiểu rõ.

## G0.16 — Recovery checkpoint drill

**Thực hiện.** Thực hành trên thay đổi test vô hại hoặc branch thử nghiệm: tạo checkpoint commit, push, tạo thay đổi sau checkpoint rồi chứng minh có thể xem/restore phiên bản trước mà không dùng force-push hoặc phá main. Sau drill, cleanup chỉ trên branch test theo cách an toàn.

**Audit bắt buộc trước khi rời phase.** Ghi commit hash recovery, remote branch và lệnh phục hồi đã kiểm chứng. Không xóa user work.

**Khi audit FAIL.** Git safety workflow chưa được coi là sẵn sàng.

## F1.1 — Pin Supabase CLI trong project + init

**Thực hiện.** Trong `D:\music\web`, cài Supabase CLI làm dev dependency bằng `npm install --save-dev supabase` (hoặc pin exact stable version sau khi đọc current Supabase skill/docs), xác nhận bằng `npx supabase --version`, sau đó chạy `npx supabase init`. Không phụ thuộc global `supabase` command. Commit `package.json`, lockfile và `supabase/config.toml`.

**Audit bắt buộc trước khi rời phase.** `npm ls supabase`, `npx supabase --version` và `npx supabase init`/config validation đều thành công. `supabase/config.toml` tồn tại và không chứa secret.

**Khi audit FAIL.** Debug npm/version compatibility; không cài một global CLI khác chỉ để né lỗi project dependency.

## F1.2 — Docker runtime + Supabase Local start

**Thực hiện.** Supabase Local cần Docker-compatible runtime. Nếu `docker version` chưa hoạt động, Codex phải thử cài/khởi tạo Docker Desktop theo tool/quyền Windows hiện có. Nếu installer cần elevation, reboot hoặc người dùng mở Docker Desktop thủ công, đây là hard external blocker: checkpoint/push và yêu cầu đúng hành động đó, KHÔNG báo project hoàn thành. Khi Docker healthy, chạy `npx supabase start`. Ghi local API URL/anon/service-role values vào `.env.local` nhưng không commit.

**Audit bắt buộc trước khi rời phase.** `docker version` thành công; `npx supabase start` health checks xanh; Studio local mở được; `.env.local` bị Git ignore. Lần đầu pull image có thể lâu và không được coi timeout ngắn là failure architecture.

**Khi audit FAIL.** Dùng systematic debugging. Chỉ dừng khi blocker thật sự cần UI/elevation/reboot ngoài tool.

## F1.3 — Tạo migration baseline rỗng có chủ đích

**Thực hiện.** Tạo migration đầu tiên dùng `npx supabase migration new bootstrap_extensions`. Migration chỉ chứa extension và schema private cần thiết cho các phase sau, không gộp cả ứng dụng vào một file khổng lồ.

**Audit bắt buộc trước khi rời phase.** Audit chạy `npx supabase db reset`. PASS khi reset từ database sạch thành công mà không cần thao tác tay.

**Khi audit FAIL.** Phase chưa hoàn thành. Codex phải ghi symptom, reproduction, expected, actual, root cause, file bị ảnh hưởng, cách sửa, regression test và residual risk vào báo cáo lỗi. Không được sang phase sau, không được tắt test, nới RLS, chuyển bucket public, thêm `any`/`@ts-ignore` hoặc catch lỗi giả thành công. Sau khi sửa phải chạy lại toàn bộ audit của phase này rồi mới đổi trạng thái sang PASS.


## F1.4 — Tạo schema private và extension

**Thực hiện.** Trong migration baseline, tạo schema `private` và extension cần thiết như `pg_trgm`, `unaccent`, `pgtap` nếu local test cần. Không grant schema private cho anon/authenticated ngoài những function execute được cho phép rõ ràng.

**Audit bắt buộc trước khi rời phase.** Audit query metadata xác nhận extension tồn tại và anon/authenticated không có quyền browse private ngoài thiết kế.

**Khi audit FAIL.** Phase chưa hoàn thành. Codex phải ghi symptom, reproduction, expected, actual, root cause, file bị ảnh hưởng, cách sửa, regression test và residual risk vào báo cáo lỗi. Không được sang phase sau, không được tắt test, nới RLS, chuyển bucket public, thêm `any`/`@ts-ignore` hoặc catch lỗi giả thành công. Sau khi sửa phải chạy lại toàn bộ audit của phase này rồi mới đổi trạng thái sang PASS.


## F1.5 — Seed framework

**Thực hiện.** Tạo `supabase/seed.sql` chỉ cho local/test. Seed không chứa dữ liệu production thật. Chuẩn bị deterministic UUID cho user A approved, user B approved, user C pending và admin test theo cách tương thích Auth local.

**Audit bắt buộc trước khi rời phase.** Audit `npx supabase db reset` hai lần liên tiếp và xác nhận seed idempotent theo kết quả mong muốn, không nhân đôi record.

**Khi audit FAIL.** Phase chưa hoàn thành. Codex phải ghi symptom, reproduction, expected, actual, root cause, file bị ảnh hưởng, cách sửa, regression test và residual risk vào báo cáo lỗi. Không được sang phase sau, không được tắt test, nới RLS, chuyển bucket public, thêm `any`/`@ts-ignore` hoặc catch lỗi giả thành công. Sau khi sửa phải chạy lại toàn bộ audit của phase này rồi mới đổi trạng thái sang PASS.


## F1.6 — Database test framework

**Thực hiện.** Tạo `supabase/tests/database` và một pgTAP smoke test kiểm tra `auth.users` hoặc extension. Không viết test giả luôn pass.

**Audit bắt buộc trước khi rời phase.** Audit bằng `npx supabase test db`. PASS khi runner thực sự tìm và chạy test.

**Khi audit FAIL.** Phase chưa hoàn thành. Codex phải ghi symptom, reproduction, expected, actual, root cause, file bị ảnh hưởng, cách sửa, regression test và residual risk vào báo cáo lỗi. Không được sang phase sau, không được tắt test, nới RLS, chuyển bucket public, thêm `any`/`@ts-ignore` hoặc catch lỗi giả thành công. Sau khi sửa phải chạy lại toàn bộ audit của phase này rồi mới đổi trạng thái sang PASS.


## F1.7 — Generate database types pipeline

**Thực hiện.** Chốt vị trí `src/types/database.types.ts`. Thêm script npm gọi `npx supabase gen types --lang typescript --local` để regenerate type sau migration.

**Audit bắt buộc trước khi rời phase.** Audit regenerate hai lần và `git diff` lần hai phải sạch. Nếu output không deterministic phải tìm nguyên nhân.

**Khi audit FAIL.** Phase chưa hoàn thành. Codex phải ghi symptom, reproduction, expected, actual, root cause, file bị ảnh hưởng, cách sửa, regression test và residual risk vào báo cáo lỗi. Không được sang phase sau, không được tắt test, nới RLS, chuyển bucket public, thêm `any`/`@ts-ignore` hoặc catch lỗi giả thành công. Sau khi sửa phải chạy lại toàn bộ audit của phase này rồi mới đổi trạng thái sang PASS.


## F1.8 — Supabase browser client

**Thực hiện.** Tạo client browser dùng anon key/public URL. Không import service role. Module client phải có tên và vị trí rõ để agent khác không tự tạo client thứ hai.

**Audit bắt buộc trước khi rời phase.** Audit grep toàn repo đảm bảo browser client không đọc service-role env.

**Khi audit FAIL.** Phase chưa hoàn thành. Codex phải ghi symptom, reproduction, expected, actual, root cause, file bị ảnh hưởng, cách sửa, regression test và residual risk vào báo cáo lỗi. Không được sang phase sau, không được tắt test, nới RLS, chuyển bucket public, thêm `any`/`@ts-ignore` hoặc catch lỗi giả thành công. Sau khi sửa phải chạy lại toàn bộ audit của phase này rồi mới đổi trạng thái sang PASS.


## F1.9 — Supabase server client

**Thực hiện.** Tạo server client theo SSR pattern hiện hành, xử lý cookie qua App Router và không tự viết session storage riêng.

**Audit bắt buộc trước khi rời phase.** Audit typecheck/build và smoke request server. PASS nếu session đọc được khi Auth local có user.

**Khi audit FAIL.** Phase chưa hoàn thành. Codex phải ghi symptom, reproduction, expected, actual, root cause, file bị ảnh hưởng, cách sửa, regression test và residual risk vào báo cáo lỗi. Không được sang phase sau, không được tắt test, nới RLS, chuyển bucket public, thêm `any`/`@ts-ignore` hoặc catch lỗi giả thành công. Sau khi sửa phải chạy lại toàn bộ audit của phase này rồi mới đổi trạng thái sang PASS.


## F1.10 — Next.js proxy session refresh

**Thực hiện.** Tạo `src/proxy.ts` theo Next.js 16 convention, dùng Supabase SSR để refresh session và matcher loại asset tĩnh cần thiết. Proxy không được coi là security boundary duy nhất; RLS vẫn là bắt buộc.

**Audit bắt buộc trước khi rời phase.** Audit anonymous request, authenticated request và static asset. PASS khi không loop redirect, không làm hỏng `/api`, build xanh.

**Khi audit FAIL.** Phase chưa hoàn thành. Codex phải ghi symptom, reproduction, expected, actual, root cause, file bị ảnh hưởng, cách sửa, regression test và residual risk vào báo cáo lỗi. Không được sang phase sau, không được tắt test, nới RLS, chuyển bucket public, thêm `any`/`@ts-ignore` hoặc catch lỗi giả thành công. Sau khi sửa phải chạy lại toàn bộ audit của phase này rồi mới đổi trạng thái sang PASS.


## F1.11 — Server-only admin client

**Thực hiện.** Tạo `src/lib/supabase/admin.ts` với `import 'server-only'`. Đây là vị trí duy nhất được phép đọc service role.

**Audit bắt buộc trước khi rời phase.** Audit grep mọi `SUPABASE_SERVICE_ROLE` và `createAdminClient`. Nếu có import chain tới `use client`, FAIL.

**Khi audit FAIL.** Phase chưa hoàn thành. Codex phải ghi symptom, reproduction, expected, actual, root cause, file bị ảnh hưởng, cách sửa, regression test và residual risk vào báo cáo lỗi. Không được sang phase sau, không được tắt test, nới RLS, chuyển bucket public, thêm `any`/`@ts-ignore` hoặc catch lỗi giả thành công. Sau khi sửa phải chạy lại toàn bộ audit của phase này rồi mới đổi trạng thái sang PASS.


## F1.12 — Local env switch

**Thực hiện.** Đảm bảo app local trỏ về Supabase Local bằng `.env.local`. Không hard-code project URL trong source. Tài liệu ghi rõ production env sẽ do Vercel cung cấp.

**Audit bắt buộc trước khi rời phase.** Audit search URL Supabase cố định trong `src`. Chỉ tài liệu/example được phép chứa placeholder.

**Khi audit FAIL.** Phase chưa hoàn thành. Codex phải ghi symptom, reproduction, expected, actual, root cause, file bị ảnh hưởng, cách sửa, regression test và residual risk vào báo cáo lỗi. Không được sang phase sau, không được tắt test, nới RLS, chuyển bucket public, thêm `any`/`@ts-ignore` hoặc catch lỗi giả thành công. Sau khi sửa phải chạy lại toàn bộ audit của phase này rồi mới đổi trạng thái sang PASS.


## F1.13 — Database reset discipline

**Thực hiện.** Thêm npm script hoặc docs command chuẩn để dev chạy `npx supabase db reset` trước database gate. Không dùng `db reset --linked` trong workflow thường ngày.

**Audit bắt buộc trước khi rời phase.** Audit command thực sự reset local và không chạm cloud. Nếu project linked, kiểm tra cờ để tránh nhầm remote.

**Khi audit FAIL.** Phase chưa hoàn thành. Codex phải ghi symptom, reproduction, expected, actual, root cause, file bị ảnh hưởng, cách sửa, regression test và residual risk vào báo cáo lỗi. Không được sang phase sau, không được tắt test, nới RLS, chuyển bucket public, thêm `any`/`@ts-ignore` hoặc catch lỗi giả thành công. Sau khi sửa phải chạy lại toàn bộ audit của phase này rồi mới đổi trạng thái sang PASS.


## F1.14 — Migration naming discipline

**Thực hiện.** Quy định mỗi thay đổi DB mới tạo migration mới có tên mô tả. Migration đã push cloud không được sửa ngược lịch sử.

**Audit bắt buộc trước khi rời phase.** Audit lịch sử migrations phải tăng tuần tự theo timestamp và tên đủ mô tả.

**Khi audit FAIL.** Phase chưa hoàn thành. Codex phải ghi symptom, reproduction, expected, actual, root cause, file bị ảnh hưởng, cách sửa, regression test và residual risk vào báo cáo lỗi. Không được sang phase sau, không được tắt test, nới RLS, chuyển bucket public, thêm `any`/`@ts-ignore` hoặc catch lỗi giả thành công. Sau khi sửa phải chạy lại toàn bộ audit của phase này rồi mới đổi trạng thái sang PASS.


## F1.15 — No declarative schemas audit

**Thực hiện.** Tìm và loại mọi `supabase/schemas` hoặc SQL tài liệu cũ đang đóng vai trò nguồn thật. Nếu cần giữ tài liệu, chỉ được link tới migration tương ứng.

**Audit bắt buộc trước khi rời phase.** Audit repo search. PASS khi migration-only là nguồn duy nhất.

**Khi audit FAIL.** Phase chưa hoàn thành. Codex phải ghi symptom, reproduction, expected, actual, root cause, file bị ảnh hưởng, cách sửa, regression test và residual risk vào báo cáo lỗi. Không được sang phase sau, không được tắt test, nới RLS, chuyển bucket public, thêm `any`/`@ts-ignore` hoặc catch lỗi giả thành công. Sau khi sửa phải chạy lại toàn bộ audit của phase này rồi mới đổi trạng thái sang PASS.


## F1.16 — Supabase local checkpoint

**Thực hiện.** Chạy từ trạng thái database đã phá bỏ hoàn toàn bằng `npx supabase db reset`, regenerate types, chạy pgTAP smoke và Next build. Sau đó commit checkpoint.

**Audit bắt buộc trước khi rời phase.** Audit tất cả lệnh xanh trong cùng một working tree. Không PASS nếu chỉ từng lệnh xanh ở các trạng thái khác nhau.

**Khi audit FAIL.** Phase chưa hoàn thành. Codex phải ghi symptom, reproduction, expected, actual, root cause, file bị ảnh hưởng, cách sửa, regression test và residual risk vào báo cáo lỗi. Không được sang phase sau, không được tắt test, nới RLS, chuyển bucket public, thêm `any`/`@ts-ignore` hoặc catch lỗi giả thành công. Sau khi sửa phải chạy lại toàn bộ audit của phase này rồi mới đổi trạng thái sang PASS.


# KHỐI D2 — DATABASE + RLS + RPC


## D2.1 — Migration profiles

**Thực hiện.** Tạo migration bảng `public.profiles` với FK `auth.users(id) on delete cascade`, email, name, avatar_url, role, access_status, is_artist, timestamps và CHECK constraints. Role/access status không được nullable.

**Audit bắt buộc trước khi rời phase.** Audit reset DB và pgTAP kiểm tra table, columns, FK, defaults, checks.

**Khi audit FAIL.** Phase chưa hoàn thành. Codex phải ghi symptom, reproduction, expected, actual, root cause, file bị ảnh hưởng, cách sửa, regression test và residual risk vào báo cáo lỗi. Không được sang phase sau, không được tắt test, nới RLS, chuyển bucket public, thêm `any`/`@ts-ignore` hoặc catch lỗi giả thành công. Sau khi sửa phải chạy lại toàn bộ audit của phase này rồi mới đổi trạng thái sang PASS.


## D2.2 — Trigger handle_new_user

**Thực hiện.** Tạo function trigger `security definer set search_path=''` insert profile khi auth user sinh ra. Mọi object phải schema-qualified.

**Audit bắt buộc trước khi rời phase.** Audit tạo user local thật và xác nhận profile đúng một row. Trigger fail phải chặn phase.

**Khi audit FAIL.** Phase chưa hoàn thành. Codex phải ghi symptom, reproduction, expected, actual, root cause, file bị ảnh hưởng, cách sửa, regression test và residual risk vào báo cáo lỗi. Không được sang phase sau, không được tắt test, nới RLS, chuyển bucket public, thêm `any`/`@ts-ignore` hoặc catch lỗi giả thành công. Sau khi sửa phải chạy lại toàn bộ audit của phase này rồi mới đổi trạng thái sang PASS.


## D2.3 — Helper private.is_approved

**Thực hiện.** Tạo `private.is_approved()` trước mọi policy cần nó. Function phải SECURITY DEFINER, stable nếu phù hợp, search_path rỗng, query schema-qualified. Revoke execute mặc định rồi chỉ grant role cần thiết.

**Audit bắt buộc trước khi rời phase.** Audit user approved trả true, pending/anon trả false và không thể đọc private schema tùy ý.

**Khi audit FAIL.** Phase chưa hoàn thành. Codex phải ghi symptom, reproduction, expected, actual, root cause, file bị ảnh hưởng, cách sửa, regression test và residual risk vào báo cáo lỗi. Không được sang phase sau, không được tắt test, nới RLS, chuyển bucket public, thêm `any`/`@ts-ignore` hoặc catch lỗi giả thành công. Sau khi sửa phải chạy lại toàn bộ audit của phase này rồi mới đổi trạng thái sang PASS.


## D2.4 — Helper private.is_admin

**Thực hiện.** Tạo `private.is_admin()` ngay sau `is_approved`, không để tới sau reports. Điều kiện admin yêu cầu cả role admin và approved.

**Audit bắt buộc trước khi rời phase.** Audit với admin/user/pending. Đây là sửa dependency blocking của plan cũ.

**Khi audit FAIL.** Phase chưa hoàn thành. Codex phải ghi symptom, reproduction, expected, actual, root cause, file bị ảnh hưởng, cách sửa, regression test và residual risk vào báo cáo lỗi. Không được sang phase sau, không được tắt test, nới RLS, chuyển bucket public, thêm `any`/`@ts-ignore` hoặc catch lỗi giả thành công. Sau khi sửa phải chạy lại toàn bộ audit của phase này rồi mới đổi trạng thái sang PASS.


## D2.5 — Profiles RLS

**Thực hiện.** Enable RLS profiles. User authenticated chỉ select row của mình. UPDATE bị giới hạn cả policy và column grant để user chỉ sửa name/avatar_url, không tự đổi role/access_status/email/is_artist.

**Audit bắt buộc trước khi rời phase.** Audit REST/client thực tế với A/B/C và thử tự nâng admin. PASS khi escalation bị từ chối.

**Khi audit FAIL.** Phase chưa hoàn thành. Codex phải ghi symptom, reproduction, expected, actual, root cause, file bị ảnh hưởng, cách sửa, regression test và residual risk vào báo cáo lỗi. Không được sang phase sau, không được tắt test, nới RLS, chuyển bucket public, thêm `any`/`@ts-ignore` hoặc catch lỗi giả thành công. Sau khi sửa phải chạy lại toàn bộ audit của phase này rồi mới đổi trạng thái sang PASS.


## D2.6 — Bootstrap admin

**Thực hiện.** Admin đầu tiên chỉ được bootstrap bằng thao tác có kiểm soát trên local/test và sau này cloud Dashboard với UUID/email đã đối chiếu. Không tạo route public bootstrap admin.

**Audit bắt buộc trước khi rời phase.** Audit codebase không có endpoint nâng admin không kiểm soát.

**Khi audit FAIL.** Phase chưa hoàn thành. Codex phải ghi symptom, reproduction, expected, actual, root cause, file bị ảnh hưởng, cách sửa, regression test và residual risk vào báo cáo lỗi. Không được sang phase sau, không được tắt test, nới RLS, chuyển bucket public, thêm `any`/`@ts-ignore` hoặc catch lỗi giả thành công. Sau khi sửa phải chạy lại toàn bộ audit của phase này rồi mới đổi trạng thái sang PASS.


## D2.7 — Migration tracks core

**Thực hiện.** Tạo tracks với owner, source, source_id, title, artist, genre, duration, storage_path, cover_url, bytes, status, license/attribution, consent_at, created_at, search_text. Không thêm `play_count`/`like_count` denormalized ở MVP để tránh stale counter.

**Audit bắt buộc trước khi rời phase.** Audit constraints cho upload/external source và pgTAP kiểm tra invalid row bị từ chối.

**Khi audit FAIL.** Phase chưa hoàn thành. Codex phải ghi symptom, reproduction, expected, actual, root cause, file bị ảnh hưởng, cách sửa, regression test và residual risk vào báo cáo lỗi. Không được sang phase sau, không được tắt test, nới RLS, chuyển bucket public, thêm `any`/`@ts-ignore` hoặc catch lỗi giả thành công. Sau khi sửa phải chạy lại toàn bộ audit của phase này rồi mới đổi trạng thái sang PASS.


## D2.8 — Track source constraints

**Thực hiện.** Upload phải có owner/storage_path/consent; nguồn ngoài phải có source_id và metadata license/attribution theo provider contract. Status chỉ pending/ready/rejected/removed.

**Audit bắt buộc trước khi rời phase.** Audit insert từng case hợp lệ và không hợp lệ. Không dùng application-only validation thay constraint.

**Khi audit FAIL.** Phase chưa hoàn thành. Codex phải ghi symptom, reproduction, expected, actual, root cause, file bị ảnh hưởng, cách sửa, regression test và residual risk vào báo cáo lỗi. Không được sang phase sau, không được tắt test, nới RLS, chuyển bucket public, thêm `any`/`@ts-ignore` hoặc catch lỗi giả thành công. Sau khi sửa phải chạy lại toàn bộ audit của phase này rồi mới đổi trạng thái sang PASS.


## D2.9 — Search normalization function

**Thực hiện.** Tạo immutable search normalize wrapper phù hợp `unaccent` và lower-case. Mọi schema reference explicit.

**Audit bắt buộc trước khi rời phase.** Audit input tiếng Việt có dấu và không dấu cho output nhất quán.

**Khi audit FAIL.** Phase chưa hoàn thành. Codex phải ghi symptom, reproduction, expected, actual, root cause, file bị ảnh hưởng, cách sửa, regression test và residual risk vào báo cáo lỗi. Không được sang phase sau, không được tắt test, nới RLS, chuyển bucket public, thêm `any`/`@ts-ignore` hoặc catch lỗi giả thành công. Sau khi sửa phải chạy lại toàn bộ audit của phase này rồi mới đổi trạng thái sang PASS.


## D2.10 — Search trigger

**Thực hiện.** Tạo trigger cập nhật `search_text` khi title/artist thay đổi. Không yêu cầu client tự gửi search_text.

**Audit bắt buộc trước khi rời phase.** Audit INSERT và UPDATE title rồi kiểm tra search_text tự đổi.

**Khi audit FAIL.** Phase chưa hoàn thành. Codex phải ghi symptom, reproduction, expected, actual, root cause, file bị ảnh hưởng, cách sửa, regression test và residual risk vào báo cáo lỗi. Không được sang phase sau, không được tắt test, nới RLS, chuyển bucket public, thêm `any`/`@ts-ignore` hoặc catch lỗi giả thành công. Sau khi sửa phải chạy lại toàn bộ audit của phase này rồi mới đổi trạng thái sang PASS.


## D2.11 — Search indexes

**Thực hiện.** Tạo GIN FTS, trigram và status index phù hợp query dự kiến. Không tạo index mà query không dùng.

**Audit bắt buộc trước khi rời phase.** Audit bằng EXPLAIN trên dataset seed đủ lớn tối thiểu để kiểm tra planner hợp lý, đồng thời ghi query mẫu vào test/docs.

**Khi audit FAIL.** Phase chưa hoàn thành. Codex phải ghi symptom, reproduction, expected, actual, root cause, file bị ảnh hưởng, cách sửa, regression test và residual risk vào báo cáo lỗi. Không được sang phase sau, không được tắt test, nới RLS, chuyển bucket public, thêm `any`/`@ts-ignore` hoặc catch lỗi giả thành công. Sau khi sửa phải chạy lại toàn bộ audit của phase này rồi mới đổi trạng thái sang PASS.


## D2.12 — Tracks RLS

**Thực hiện.** Approved user chỉ đọc track ready. Owner approved được đọc track pending/rejected của chính mình và chỉ thao tác row của mình trong phạm vi nghiệp vụ. Anon/pending không đọc catalog private.

**Audit bắt buộc trước khi rời phase.** Audit matrix A/B/C/anon bằng pgTAP và client integration.

**Khi audit FAIL.** Phase chưa hoàn thành. Codex phải ghi symptom, reproduction, expected, actual, root cause, file bị ảnh hưởng, cách sửa, regression test và residual risk vào báo cáo lỗi. Không được sang phase sau, không được tắt test, nới RLS, chuyển bucket public, thêm `any`/`@ts-ignore` hoặc catch lỗi giả thành công. Sau khi sửa phải chạy lại toàn bộ audit của phase này rồi mới đổi trạng thái sang PASS.


## D2.13 — Migration playlists

**Thực hiện.** Tạo playlists owner, name, is_public và timestamps. 'Public' chỉ nghĩa chia sẻ trong nhóm approved, không phải Internet.

**Audit bắt buộc trước khi rời phase.** Audit private playlist của A không đọc được bởi B; public playlist đọc được bởi B approved; C pending bị chặn.

**Khi audit FAIL.** Phase chưa hoàn thành. Codex phải ghi symptom, reproduction, expected, actual, root cause, file bị ảnh hưởng, cách sửa, regression test và residual risk vào báo cáo lỗi. Không được sang phase sau, không được tắt test, nới RLS, chuyển bucket public, thêm `any`/`@ts-ignore` hoặc catch lỗi giả thành công. Sau khi sửa phải chạy lại toàn bộ audit của phase này rồi mới đổi trạng thái sang PASS.


## D2.14 — Migration playlist_tracks

**Thực hiện.** Tạo join table với PK `(playlist_id, track_id)`, position >=0 và unique `(playlist_id,position)` DEFERRABLE. Cả FK dùng cascade phù hợp.

**Audit bắt buộc trước khi rời phase.** Audit duplicate track và duplicate position bị chặn theo transaction semantics mong muốn.

**Khi audit FAIL.** Phase chưa hoàn thành. Codex phải ghi symptom, reproduction, expected, actual, root cause, file bị ảnh hưởng, cách sửa, regression test và residual risk vào báo cáo lỗi. Không được sang phase sau, không được tắt test, nới RLS, chuyển bucket public, thêm `any`/`@ts-ignore` hoặc catch lỗi giả thành công. Sau khi sửa phải chạy lại toàn bộ audit của phase này rồi mới đổi trạng thái sang PASS.


## D2.15 — playlist_tracks RLS

**Thực hiện.** Policy phải dựa vào playlist ownership/public visibility và track ready. Không grant authenticated đọc tất cả join table.

**Audit bắt buộc trước khi rời phase.** Audit B không suy ra nội dung playlist private của A.

**Khi audit FAIL.** Phase chưa hoàn thành. Codex phải ghi symptom, reproduction, expected, actual, root cause, file bị ảnh hưởng, cách sửa, regression test và residual risk vào báo cáo lỗi. Không được sang phase sau, không được tắt test, nới RLS, chuyển bucket public, thêm `any`/`@ts-ignore` hoặc catch lỗi giả thành công. Sau khi sửa phải chạy lại toàn bộ audit của phase này rồi mới đổi trạng thái sang PASS.


## D2.16 — RPC append_playlist_track

**Thực hiện.** Tạo RPC transaction thêm track ở cuối playlist và tính position atomic. Function phải xác nhận caller là owner và track được phép dùng. Không dùng client `MAX(position)+1`.

**Audit bắt buộc trước khi rời phase.** Audit hai request đồng thời không sinh duplicate position hoặc mất row.

**Khi audit FAIL.** Phase chưa hoàn thành. Codex phải ghi symptom, reproduction, expected, actual, root cause, file bị ảnh hưởng, cách sửa, regression test và residual risk vào báo cáo lỗi. Không được sang phase sau, không được tắt test, nới RLS, chuyển bucket public, thêm `any`/`@ts-ignore` hoặc catch lỗi giả thành công. Sau khi sửa phải chạy lại toàn bộ audit của phase này rồi mới đổi trạng thái sang PASS.


## D2.17 — RPC reorder_playlist_track

**Thực hiện.** Tạo RPC transaction đổi position dùng constraint DEFERRABLE. Không cập nhật từng row rời rạc từ client.

**Audit bắt buộc trước khi rời phase.** Audit reorder đồng thời bằng hai session/requests; invariant position vẫn đúng.

**Khi audit FAIL.** Phase chưa hoàn thành. Codex phải ghi symptom, reproduction, expected, actual, root cause, file bị ảnh hưởng, cách sửa, regression test và residual risk vào báo cáo lỗi. Không được sang phase sau, không được tắt test, nới RLS, chuyển bucket public, thêm `any`/`@ts-ignore` hoặc catch lỗi giả thành công. Sau khi sửa phải chạy lại toàn bộ audit của phase này rồi mới đổi trạng thái sang PASS.


## D2.18 — Migration likes

**Thực hiện.** Tạo likes PK `(user_id,track_id)` với cascade và RLS user chỉ quản lý row mình.

**Audit bắt buộc trước khi rời phase.** Audit user B không đọc danh sách like riêng của A nếu không có requirement chia sẻ.

**Khi audit FAIL.** Phase chưa hoàn thành. Codex phải ghi symptom, reproduction, expected, actual, root cause, file bị ảnh hưởng, cách sửa, regression test và residual risk vào báo cáo lỗi. Không được sang phase sau, không được tắt test, nới RLS, chuyển bucket public, thêm `any`/`@ts-ignore` hoặc catch lỗi giả thành công. Sau khi sửa phải chạy lại toàn bộ audit của phase này rồi mới đổi trạng thái sang PASS.


## D2.19 — RPC toggle_like atomic

**Thực hiện.** Tạo RPC atomic toggle like dựa trên auth.uid, không nhận user_id từ client. Unique conflict phải được xử lý trong transaction thay vì request race ngoài app.

**Audit bắt buộc trước khi rời phase.** Audit hai request gần đồng thời; kết quả cuối xác định được và DB không lỗi invariant.

**Khi audit FAIL.** Phase chưa hoàn thành. Codex phải ghi symptom, reproduction, expected, actual, root cause, file bị ảnh hưởng, cách sửa, regression test và residual risk vào báo cáo lỗi. Không được sang phase sau, không được tắt test, nới RLS, chuyển bucket public, thêm `any`/`@ts-ignore` hoặc catch lỗi giả thành công. Sau khi sửa phải chạy lại toàn bộ audit của phase này rồi mới đổi trạng thái sang PASS.


## D2.20 — Migration play_history

**Thực hiện.** Tạo history identity PK, user_id, track_id, played_at và index user-time/track-time. RLS user chỉ đọc/ghi lịch sử của mình.

**Audit bắt buộc trước khi rời phase.** Audit query mới nhất và negative cross-user.

**Khi audit FAIL.** Phase chưa hoàn thành. Codex phải ghi symptom, reproduction, expected, actual, root cause, file bị ảnh hưởng, cách sửa, regression test và residual risk vào báo cáo lỗi. Không được sang phase sau, không được tắt test, nới RLS, chuyển bucket public, thêm `any`/`@ts-ignore` hoặc catch lỗi giả thành công. Sau khi sửa phải chạy lại toàn bộ audit của phase này rồi mới đổi trạng thái sang PASS.


## D2.21 — Migration reports

**Thực hiện.** Tạo reports với unique reporter-track, reason length check, status. Policy insert cho approved user row mình; admin read/update dùng `private.is_admin()` đã tồn tại từ D2.4.

**Audit bắt buộc trước khi rời phase.** Audit migration chạy từ đầu không còn lỗi dependency `is_admin` như plan cũ.

**Khi audit FAIL.** Phase chưa hoàn thành. Codex phải ghi symptom, reproduction, expected, actual, root cause, file bị ảnh hưởng, cách sửa, regression test và residual risk vào báo cáo lỗi. Không được sang phase sau, không được tắt test, nới RLS, chuyển bucket public, thêm `any`/`@ts-ignore` hoặc catch lỗi giả thành công. Sau khi sửa phải chạy lại toàn bộ audit của phase này rồi mới đổi trạng thái sang PASS.


## D2.22 — Migration audit_log

**Thực hiện.** Tạo audit_log với `admin_id` nullable FK profiles `ON DELETE SET NULL`, action, target, success, metadata tối thiểu và timestamp. Chỉ admin được đọc.

**Audit bắt buộc trước khi rời phase.** Audit xóa admin test không bị FK audit_log chặn và log lịch sử vẫn còn.

**Khi audit FAIL.** Phase chưa hoàn thành. Codex phải ghi symptom, reproduction, expected, actual, root cause, file bị ảnh hưởng, cách sửa, regression test và residual risk vào báo cáo lỗi. Không được sang phase sau, không được tắt test, nới RLS, chuyển bucket public, thêm `any`/`@ts-ignore` hoặc catch lỗi giả thành công. Sau khi sửa phải chạy lại toàn bộ audit của phase này rồi mới đổi trạng thái sang PASS.


## D2.23 — Migration upload_reservations

**Thực hiện.** Tạo bảng reservation gồm id, owner_id, object_path, declared_bytes, mime, consent_at, expires_at, consumed_at, status và timestamps. Trạng thái phải có CHECK rõ ràng.

**Audit bắt buộc trước khi rời phase.** Audit reservation hết hạn/consumed không thể bị consume sai lần nữa.

**Khi audit FAIL.** Phase chưa hoàn thành. Codex phải ghi symptom, reproduction, expected, actual, root cause, file bị ảnh hưởng, cách sửa, regression test và residual risk vào báo cáo lỗi. Không được sang phase sau, không được tắt test, nới RLS, chuyển bucket public, thêm `any`/`@ts-ignore` hoặc catch lỗi giả thành công. Sau khi sửa phải chạy lại toàn bộ audit của phase này rồi mới đổi trạng thái sang PASS.


## D2.24 — RPC reserve_upload_quota

**Thực hiện.** Tạo RPC transaction tính usage hiện tại cộng reservation chưa hết hạn rồi quyết định quota. Giới hạn file 20MB và tổng 100MB được enforce phía server. Không dùng SELECT rồi INSERT ở hai request tách rời.

**Audit bắt buộc trước khi rời phase.** Audit hai request song song sát quota; tổng reserved+used không vượt 100MB.

**Khi audit FAIL.** Phase chưa hoàn thành. Codex phải ghi symptom, reproduction, expected, actual, root cause, file bị ảnh hưởng, cách sửa, regression test và residual risk vào báo cáo lỗi. Không được sang phase sau, không được tắt test, nới RLS, chuyển bucket public, thêm `any`/`@ts-ignore` hoặc catch lỗi giả thành công. Sau khi sửa phải chạy lại toàn bộ audit của phase này rồi mới đổi trạng thái sang PASS.


## D2.25 — Usage view

**Thực hiện.** Tạo view hoặc RPC usage chỉ được server đọc sau khi xác thực user; không expose toàn bộ usage của các owner khác. User chưa upload phải thấy 0.

**Audit bắt buộc trước khi rời phase.** Audit A không truy xuất usage B qua Data API.

**Khi audit FAIL.** Phase chưa hoàn thành. Codex phải ghi symptom, reproduction, expected, actual, root cause, file bị ảnh hưởng, cách sửa, regression test và residual risk vào báo cáo lỗi. Không được sang phase sau, không được tắt test, nới RLS, chuyển bucket public, thêm `any`/`@ts-ignore` hoặc catch lỗi giả thành công. Sau khi sửa phải chạy lại toàn bộ audit của phase này rồi mới đổi trạng thái sang PASS.


## D2.26 — Trending view

**Thực hiện.** Tạo regular view trending từ ready tracks và play_history 7 ngày. View không grant trực tiếp cho anon/authenticated nếu API server sẽ query bằng service role sau check approved.

**Audit bắt buộc trước khi rời phase.** Audit score order trên seed deterministic và xác nhận không cần REFRESH cho regular view.

**Khi audit FAIL.** Phase chưa hoàn thành. Codex phải ghi symptom, reproduction, expected, actual, root cause, file bị ảnh hưởng, cách sửa, regression test và residual risk vào báo cáo lỗi. Không được sang phase sau, không được tắt test, nới RLS, chuyển bucket public, thêm `any`/`@ts-ignore` hoặc catch lỗi giả thành công. Sau khi sửa phải chạy lại toàn bộ audit của phase này rồi mới đổi trạng thái sang PASS.


## D2.27 — Maintenance primitives

**Thực hiện.** Tạo SQL/RPC cần thiết để xóa history >90 ngày và reservation hết hạn. Không xóa Storage object bằng trigger SQL mạng; cleanup object phải qua server/Storage API.

**Audit bắt buộc trước khi rời phase.** Audit seed row 91 ngày bị dọn, row mới vẫn còn.

**Khi audit FAIL.** Phase chưa hoàn thành. Codex phải ghi symptom, reproduction, expected, actual, root cause, file bị ảnh hưởng, cách sửa, regression test và residual risk vào báo cáo lỗi. Không được sang phase sau, không được tắt test, nới RLS, chuyển bucket public, thêm `any`/`@ts-ignore` hoặc catch lỗi giả thành công. Sau khi sửa phải chạy lại toàn bộ audit của phase này rồi mới đổi trạng thái sang PASS.


## D2.28 — Full database pgTAP suite

**Thực hiện.** Viết pgTAP cho schema, constraints, helpers, RLS và RPC quan trọng với A/B/C/admin/anon. Mỗi policy phải có allow test và deny test.

**Audit bắt buộc trước khi rời phase.** Audit `npx supabase db reset` rồi `npx supabase test db`. PASS chỉ khi toàn bộ suite xanh.

**Khi audit FAIL.** Phase chưa hoàn thành. Codex phải ghi symptom, reproduction, expected, actual, root cause, file bị ảnh hưởng, cách sửa, regression test và residual risk vào báo cáo lỗi. Không được sang phase sau, không được tắt test, nới RLS, chuyển bucket public, thêm `any`/`@ts-ignore` hoặc catch lỗi giả thành công. Sau khi sửa phải chạy lại toàn bộ audit của phase này rồi mới đổi trạng thái sang PASS.


## D2.29 — Generate types after DB complete

**Thực hiện.** Regenerate `database.types.ts` từ local database sau toàn bộ D2 migrations. Sửa code dùng type cũ nếu có.

**Audit bắt buộc trước khi rời phase.** Audit regenerate lần hai không tạo diff và TypeScript xanh.

**Khi audit FAIL.** Phase chưa hoàn thành. Codex phải ghi symptom, reproduction, expected, actual, root cause, file bị ảnh hưởng, cách sửa, regression test và residual risk vào báo cáo lỗi. Không được sang phase sau, không được tắt test, nới RLS, chuyển bucket public, thêm `any`/`@ts-ignore` hoặc catch lỗi giả thành công. Sau khi sửa phải chạy lại toàn bộ audit của phase này rồi mới đổi trạng thái sang PASS.


## D2.30 — Database reproducibility gate

**Thực hiện.** Xóa database local bằng reset, không mở Studio để sửa tay, chạy migrations+seed+tests từ đầu. Đây là gate chứng minh migration-only thực sự đủ.

**Audit bắt buộc trước khi rời phase.** Audit bắt buộc lưu output reset/test. Nếu phải gõ SQL tay để cứu DB thì quay lại migration tương ứng, phase FAIL.

**Khi audit FAIL.** Phase chưa hoàn thành. Codex phải ghi symptom, reproduction, expected, actual, root cause, file bị ảnh hưởng, cách sửa, regression test và residual risk vào báo cáo lỗi. Không được sang phase sau, không được tắt test, nới RLS, chuyển bucket public, thêm `any`/`@ts-ignore` hoặc catch lỗi giả thành công. Sau khi sửa phải chạy lại toàn bộ audit của phase này rồi mới đổi trạng thái sang PASS.


# KHỐI S3 — AUTH + STORAGE + UPLOAD SECURITY


## S3.1 — Invite-only Auth local

**Thực hiện.** Khóa open signup theo config phù hợp. Tạo flow email/password cho user đã được invite/chuẩn bị trước. Google OAuth chưa bật nếu allowlist callback chưa hoàn tất.

**Audit bắt buộc trước khi rời phase.** Audit anonymous không tự đăng ký được bằng UI/API ngoài flow admin dự kiến.

**Khi audit FAIL.** Phase chưa hoàn thành. Codex phải ghi symptom, reproduction, expected, actual, root cause, file bị ảnh hưởng, cách sửa, regression test và residual risk vào báo cáo lỗi. Không được sang phase sau, không được tắt test, nới RLS, chuyển bucket public, thêm `any`/`@ts-ignore` hoặc catch lỗi giả thành công. Sau khi sửa phải chạy lại toàn bộ audit của phase này rồi mới đổi trạng thái sang PASS.


## S3.2 — Login UI

**Thực hiện.** Tạo `/login` xử lý login, error tiếng Việt vừa đủ và redirect sau success. Không log password/token.

**Audit bắt buộc trước khi rời phase.** Audit wrong password, pending user, approved user và session reload.

**Khi audit FAIL.** Phase chưa hoàn thành. Codex phải ghi symptom, reproduction, expected, actual, root cause, file bị ảnh hưởng, cách sửa, regression test và residual risk vào báo cáo lỗi. Không được sang phase sau, không được tắt test, nới RLS, chuyển bucket public, thêm `any`/`@ts-ignore` hoặc catch lỗi giả thành công. Sau khi sửa phải chạy lại toàn bộ audit của phase này rồi mới đổi trạng thái sang PASS.


## S3.3 — Logout

**Thực hiện.** Tạo logout server/client phù hợp SDK. Xóa session local đúng cách và không giữ protected cache.

**Audit bắt buộc trước khi rời phase.** Audit logout rồi back/refresh không xem dữ liệu protected.

**Khi audit FAIL.** Phase chưa hoàn thành. Codex phải ghi symptom, reproduction, expected, actual, root cause, file bị ảnh hưởng, cách sửa, regression test và residual risk vào báo cáo lỗi. Không được sang phase sau, không được tắt test, nới RLS, chuyển bucket public, thêm `any`/`@ts-ignore` hoặc catch lỗi giả thành công. Sau khi sửa phải chạy lại toàn bộ audit của phase này rồi mới đổi trạng thái sang PASS.


## S3.4 — Approved gate server

**Thực hiện.** Tạo helper server kiểm tra session và approval để route/page dùng thống nhất. Helper không thay RLS mà chỉ cải thiện UX/early reject.

**Audit bắt buộc trước khi rời phase.** Audit pending user bị 403/redirect ở API/UI và vẫn bị RLS chặn khi gọi trực tiếp DB.

**Khi audit FAIL.** Phase chưa hoàn thành. Codex phải ghi symptom, reproduction, expected, actual, root cause, file bị ảnh hưởng, cách sửa, regression test và residual risk vào báo cáo lỗi. Không được sang phase sau, không được tắt test, nới RLS, chuyển bucket public, thêm `any`/`@ts-ignore` hoặc catch lỗi giả thành công. Sau khi sửa phải chạy lại toàn bộ audit của phase này rồi mới đổi trạng thái sang PASS.


## S3.5 — Admin gate server

**Thực hiện.** Tạo helper admin dựa trên DB helper/profile đáng tin, dùng trong route/page admin. Không lấy role từ client payload.

**Audit bắt buộc trước khi rời phase.** Audit user sửa localStorage/client state thành admin vẫn không qua gate.

**Khi audit FAIL.** Phase chưa hoàn thành. Codex phải ghi symptom, reproduction, expected, actual, root cause, file bị ảnh hưởng, cách sửa, regression test và residual risk vào báo cáo lỗi. Không được sang phase sau, không được tắt test, nới RLS, chuyển bucket public, thêm `any`/`@ts-ignore` hoặc catch lỗi giả thành công. Sau khi sửa phải chạy lại toàn bộ audit của phase này rồi mới đổi trạng thái sang PASS.


## S3.6 — Buckets private

**Thực hiện.** Tạo bucket `audio` và `covers` private trong migration/seed-compatible config hoặc migration SQL theo khả năng Supabase hiện hành. Không đổi public để né signed URL.

**Audit bắt buộc trước khi rời phase.** Audit anon download object thất bại.

**Khi audit FAIL.** Phase chưa hoàn thành. Codex phải ghi symptom, reproduction, expected, actual, root cause, file bị ảnh hưởng, cách sửa, regression test và residual risk vào báo cáo lỗi. Không được sang phase sau, không được tắt test, nới RLS, chuyển bucket public, thêm `any`/`@ts-ignore` hoặc catch lỗi giả thành công. Sau khi sửa phải chạy lại toàn bộ audit của phase này rồi mới đổi trạng thái sang PASS.


## S3.7 — Storage policies

**Thực hiện.** Policy object kiểm tra bucket, auth uid và path server-generated. Không tin filename/path client. Nếu Storage migration cần object schema changes, phải theo phương thức Supabase hỗ trợ và reset được local.

**Audit bắt buộc trước khi rời phase.** Audit A/B/anon matrix cho upload/read/delete.

**Khi audit FAIL.** Phase chưa hoàn thành. Codex phải ghi symptom, reproduction, expected, actual, root cause, file bị ảnh hưởng, cách sửa, regression test và residual risk vào báo cáo lỗi. Không được sang phase sau, không được tắt test, nới RLS, chuyển bucket public, thêm `any`/`@ts-ignore` hoặc catch lỗi giả thành công. Sau khi sửa phải chạy lại toàn bộ audit của phase này rồi mới đổi trạng thái sang PASS.


## S3.8 — Upload sign API

**Thực hiện.** Tạo `POST /api/upload/sign` với Zod cho filename,size,mime, consentAccepted. Server gọi quota reservation RPC, sinh UUID object path và trả signed upload capability.

**Audit bắt buộc trước khi rời phase.** Audit exe/path traversal/oversize/no-consent bị từ chối và không tạo reservation rác.

**Khi audit FAIL.** Phase chưa hoàn thành. Codex phải ghi symptom, reproduction, expected, actual, root cause, file bị ảnh hưởng, cách sửa, regression test và residual risk vào báo cáo lỗi. Không được sang phase sau, không được tắt test, nới RLS, chuyển bucket public, thêm `any`/`@ts-ignore` hoặc catch lỗi giả thành công. Sau khi sửa phải chạy lại toàn bộ audit của phase này rồi mới đổi trạng thái sang PASS.


## S3.9 — Signed upload constraints

**Thực hiện.** Signed upload phải ràng buộc object path/content type trong mức SDK cho phép. Không cho client tự thay owner folder.

**Audit bắt buộc trước khi rời phase.** Audit cố upload object khác path reservation và xác nhận thất bại hoặc complete không chấp nhận.

**Khi audit FAIL.** Phase chưa hoàn thành. Codex phải ghi symptom, reproduction, expected, actual, root cause, file bị ảnh hưởng, cách sửa, regression test và residual risk vào báo cáo lỗi. Không được sang phase sau, không được tắt test, nới RLS, chuyển bucket public, thêm `any`/`@ts-ignore` hoặc catch lỗi giả thành công. Sau khi sửa phải chạy lại toàn bộ audit của phase này rồi mới đổi trạng thái sang PASS.


## S3.10 — Upload complete verifier

**Thực hiện.** Tạo `/api/upload/complete` chỉ nhận reservationId. Server xác nhận reservation owner, object tồn tại, actual size, magic bytes, metadata/duration và status chưa consumed.

**Audit bắt buộc trước khi rời phase.** Audit fake MIME, object thiếu, size lệch và complete lặp.

**Khi audit FAIL.** Phase chưa hoàn thành. Codex phải ghi symptom, reproduction, expected, actual, root cause, file bị ảnh hưởng, cách sửa, regression test và residual risk vào báo cáo lỗi. Không được sang phase sau, không được tắt test, nới RLS, chuyển bucket public, thêm `any`/`@ts-ignore` hoặc catch lỗi giả thành công. Sau khi sửa phải chạy lại toàn bộ audit của phase này rồi mới đổi trạng thái sang PASS.


## S3.11 — Consent propagation

**Thực hiện.** Consent timestamp được tạo server-side ở reservation khi client xác nhận checkbox; complete copy timestamp đó vào track. Không nhận timestamp tự do từ client.

**Audit bắt buộc trước khi rời phase.** Audit track upload hợp lệ luôn có consent_at khớp reservation.

**Khi audit FAIL.** Phase chưa hoàn thành. Codex phải ghi symptom, reproduction, expected, actual, root cause, file bị ảnh hưởng, cách sửa, regression test và residual risk vào báo cáo lỗi. Không được sang phase sau, không được tắt test, nới RLS, chuyển bucket public, thêm `any`/`@ts-ignore` hoặc catch lỗi giả thành công. Sau khi sửa phải chạy lại toàn bộ audit của phase này rồi mới đổi trạng thái sang PASS.


## S3.12 — Audio hash dedup

**Thực hiện.** Thêm migration cột hash và partial unique index `(owner,audio_hash) WHERE source='upload'`. Hash tính server-side trong quá trình verify, không tin client.

**Audit bắt buộc trước khi rời phase.** Audit hai upload cùng file và hai complete cạnh tranh chỉ còn một track/object hợp lệ.

**Khi audit FAIL.** Phase chưa hoàn thành. Codex phải ghi symptom, reproduction, expected, actual, root cause, file bị ảnh hưởng, cách sửa, regression test và residual risk vào báo cáo lỗi. Không được sang phase sau, không được tắt test, nới RLS, chuyển bucket public, thêm `any`/`@ts-ignore` hoặc catch lỗi giả thành công. Sau khi sửa phải chạy lại toàn bộ audit của phase này rồi mới đổi trạng thái sang PASS.


## S3.13 — Invalid upload cleanup

**Thực hiện.** Khi verify fail hoặc dedup conflict, xóa object mới và cập nhật reservation trạng thái failure/cleanup. Cleanup phải idempotent.

**Audit bắt buộc trước khi rời phase.** Audit request lặp không xóa nhầm object đã hợp lệ.

**Khi audit FAIL.** Phase chưa hoàn thành. Codex phải ghi symptom, reproduction, expected, actual, root cause, file bị ảnh hưởng, cách sửa, regression test và residual risk vào báo cáo lỗi. Không được sang phase sau, không được tắt test, nới RLS, chuyển bucket public, thêm `any`/`@ts-ignore` hoặc catch lỗi giả thành công. Sau khi sửa phải chạy lại toàn bộ audit của phase này rồi mới đổi trạng thái sang PASS.


## S3.14 — Account deletion design

**Thực hiện.** Tạo flow re-auth gần đây, xác nhận rõ, server liệt kê/xóa Storage object qua Storage API, sau đó xóa auth user để cascade DB. Không DELETE metadata trực tiếp trong schema storage.

**Audit bắt buộc trước khi rời phase.** Audit bằng tài khoản test có playlist/audio/report/history; sau xóa login thất bại và không còn object owner.

**Khi audit FAIL.** Phase chưa hoàn thành. Codex phải ghi symptom, reproduction, expected, actual, root cause, file bị ảnh hưởng, cách sửa, regression test và residual risk vào báo cáo lỗi. Không được sang phase sau, không được tắt test, nới RLS, chuyển bucket public, thêm `any`/`@ts-ignore` hoặc catch lỗi giả thành công. Sau khi sửa phải chạy lại toàn bộ audit của phase này rồi mới đổi trạng thái sang PASS.


## S3.15 — Service-role leak audit

**Thực hiện.** Grep toàn bộ source, build output và client static bundle cho tên/value service role và bearer token. Nếu secret xuất hiện ở `.next/static`, phase FAIL blocking.

**Audit bắt buộc trước khi rời phase.** Audit phải dùng cả source grep và build artifact grep.

**Khi audit FAIL.** Phase chưa hoàn thành. Codex phải ghi symptom, reproduction, expected, actual, root cause, file bị ảnh hưởng, cách sửa, regression test và residual risk vào báo cáo lỗi. Không được sang phase sau, không được tắt test, nới RLS, chuyển bucket public, thêm `any`/`@ts-ignore` hoặc catch lỗi giả thành công. Sau khi sửa phải chạy lại toàn bộ audit của phase này rồi mới đổi trạng thái sang PASS.


## S3.16 — Basic security headers

**Thực hiện.** Cấu hình `X-Content-Type-Options`, frame policy phù hợp, referrer policy và các header an toàn không phá audio. CSP được deferred nếu chưa đủ test nhưng phải ghi accepted risk.

**Audit bắt buộc trước khi rời phase.** Audit `curl -I` local/preview và browser smoke.

**Khi audit FAIL.** Phase chưa hoàn thành. Codex phải ghi symptom, reproduction, expected, actual, root cause, file bị ảnh hưởng, cách sửa, regression test và residual risk vào báo cáo lỗi. Không được sang phase sau, không được tắt test, nới RLS, chuyển bucket public, thêm `any`/`@ts-ignore` hoặc catch lỗi giả thành công. Sau khi sửa phải chạy lại toàn bộ audit của phase này rồi mới đổi trạng thái sang PASS.


## S3.17 — Structured error response

**Thực hiện.** Tạo requestId cho API error, log server structured và redact token/secret/PII. Client chỉ nhận error code/message/requestId, không stack.

**Audit bắt buộc trước khi rời phase.** Audit ép 500 và tìm secret giả trong response/log client.

**Khi audit FAIL.** Phase chưa hoàn thành. Codex phải ghi symptom, reproduction, expected, actual, root cause, file bị ảnh hưởng, cách sửa, regression test và residual risk vào báo cáo lỗi. Không được sang phase sau, không được tắt test, nới RLS, chuyển bucket public, thêm `any`/`@ts-ignore` hoặc catch lỗi giả thành công. Sau khi sửa phải chạy lại toàn bộ audit của phase này rồi mới đổi trạng thái sang PASS.


## S3.18 — Security gate Auth+Storage

**Thực hiện.** Chạy toàn matrix auth/RLS/storage/upload abuse trước khi sang catalog. Không cho phép redirect UI được dùng làm bằng chứng security.

**Audit bắt buộc trước khi rời phase.** Audit lưu evidence của anon, pending, approved A/B, admin và path traversal/fake MIME.

**Khi audit FAIL.** Phase chưa hoàn thành. Codex phải ghi symptom, reproduction, expected, actual, root cause, file bị ảnh hưởng, cách sửa, regression test và residual risk vào báo cáo lỗi. Không được sang phase sau, không được tắt test, nới RLS, chuyển bucket public, thêm `any`/`@ts-ignore` hoặc catch lỗi giả thành công. Sau khi sửa phải chạy lại toàn bộ audit của phase này rồi mới đổi trạng thái sang PASS.


# KHỐI M4 — CATALOG NGOÀI + STREAM + PLAYER


## M4.1 — Track DTO canonical

**Thực hiện.** Tạo `src/types/track.ts` làm DTO duy nhất cho UI/player với source upload/audius/jamendo, id,title,artist,stream/resolve metadata, cover,duration,license/attribution. DB snake_case chỉ map tại server boundary.

**Audit bắt buộc trước khi rời phase.** Audit grep không có DTO Track thứ hai tự phát.

**Khi audit FAIL.** Phase chưa hoàn thành. Codex phải ghi symptom, reproduction, expected, actual, root cause, file bị ảnh hưởng, cách sửa, regression test và residual risk vào báo cáo lỗi. Không được sang phase sau, không được tắt test, nới RLS, chuyển bucket public, thêm `any`/`@ts-ignore` hoặc catch lỗi giả thành công. Sau khi sửa phải chạy lại toàn bộ audit của phase này rồi mới đổi trạng thái sang PASS.


## M4.2 — QueryProvider

**Thực hiện.** Tạo TanStack Query provider một instance, staleTime khoảng 5 phút cho catalog phù hợp MVP, retry có giới hạn và không tạo QueryClient mỗi render.

**Audit bắt buộc trước khi rời phase.** Audit React render và typecheck.

**Khi audit FAIL.** Phase chưa hoàn thành. Codex phải ghi symptom, reproduction, expected, actual, root cause, file bị ảnh hưởng, cách sửa, regression test và residual risk vào báo cáo lỗi. Không được sang phase sau, không được tắt test, nới RLS, chuyển bucket public, thêm `any`/`@ts-ignore` hoặc catch lỗi giả thành công. Sau khi sửa phải chạy lại toàn bộ audit của phase này rồi mới đổi trạng thái sang PASS.


## M4.3 — Player store

**Thực hiện.** Tạo Zustand store ở `src/stores/player-store.ts`; persist chỉ state serializable như queue,index,repeat,shuffle,volume,muted. Không persist Howl/currentTime/isPlaying.

**Audit bắt buộc trước khi rời phase.** Audit reload nhiều lần và grep không có store trùng ở `src/lib`.

**Khi audit FAIL.** Phase chưa hoàn thành. Codex phải ghi symptom, reproduction, expected, actual, root cause, file bị ảnh hưởng, cách sửa, regression test và residual risk vào báo cáo lỗi. Không được sang phase sau, không được tắt test, nới RLS, chuyển bucket public, thêm `any`/`@ts-ignore` hoặc catch lỗi giả thành công. Sau khi sửa phải chạy lại toàn bộ audit của phase này rồi mới đổi trạng thái sang PASS.


## M4.4 — Jamendo server adapter

**Thực hiện.** Tạo server-only adapter đọc credential từ env, timeout và normalize response về Track DTO. Không import từ client.

**Audit bắt buộc trước khi rời phase.** Audit thiếu key trả lỗi kiểm soát, key không xuất hiện Network/client bundle.

**Khi audit FAIL.** Phase chưa hoàn thành. Codex phải ghi symptom, reproduction, expected, actual, root cause, file bị ảnh hưởng, cách sửa, regression test và residual risk vào báo cáo lỗi. Không được sang phase sau, không được tắt test, nới RLS, chuyển bucket public, thêm `any`/`@ts-ignore` hoặc catch lỗi giả thành công. Sau khi sửa phải chạy lại toàn bộ audit của phase này rồi mới đổi trạng thái sang PASS.


## M4.5 — Audius credentials

**Thực hiện.** Thêm env cho Audius API key và bearer token khi backend flow yêu cầu. Không dùng discovery provider anonymous cũ. Bearer token backend-only.

**Audit bắt buộc trước khi rời phase.** Audit docs/env và bundle leak trước khi viết adapter.

**Khi audit FAIL.** Phase chưa hoàn thành. Codex phải ghi symptom, reproduction, expected, actual, root cause, file bị ảnh hưởng, cách sửa, regression test và residual risk vào báo cáo lỗi. Không được sang phase sau, không được tắt test, nới RLS, chuyển bucket public, thêm `any`/`@ts-ignore` hoặc catch lỗi giả thành công. Sau khi sửa phải chạy lại toàn bộ audit của phase này rồi mới đổi trạng thái sang PASS.


## M4.6 — Audius server adapter

**Thực hiện.** Dùng SDK/API hiện hành để search/trending/resolve stream theo credential chính thức, timeout rõ ràng và normalize DTO. Không hard-code discovery host đã lỗi thời.

**Audit bắt buộc trước khi rời phase.** Audit request thật hoặc mock contract; auth failure phải được report rõ.

**Khi audit FAIL.** Phase chưa hoàn thành. Codex phải ghi symptom, reproduction, expected, actual, root cause, file bị ảnh hưởng, cách sửa, regression test và residual risk vào báo cáo lỗi. Không được sang phase sau, không được tắt test, nới RLS, chuyển bucket public, thêm `any`/`@ts-ignore` hoặc catch lỗi giả thành công. Sau khi sửa phải chạy lại toàn bộ audit của phase này rồi mới đổi trạng thái sang PASS.


## M4.7 — Provider compatibility spike trước player

**Thực hiện.** Trước khi xây UI player, test mỗi nguồn: metadata, URL resolution, playback, Range/seek/CORS trên browser và ghi `docs/compatibility.md`. Nguồn không đạt phải có fallback hoặc bị disable có chủ đích.

**Audit bắt buộc trước khi rời phase.** Audit đây là blocking gate. Không được tiếp tục player với giả định 'chắc sẽ chạy'.

**Khi audit FAIL.** Phase chưa hoàn thành. Codex phải ghi symptom, reproduction, expected, actual, root cause, file bị ảnh hưởng, cách sửa, regression test và residual risk vào báo cáo lỗi. Không được sang phase sau, không được tắt test, nới RLS, chuyển bucket public, thêm `any`/`@ts-ignore` hoặc catch lỗi giả thành công. Sau khi sửa phải chạy lại toàn bộ audit của phase này rồi mới đổi trạng thái sang PASS.


## M4.8 — Unified stream resolver API

**Thực hiện.** Tạo `/api/stream` nhận source + canonical track id. Upload trả Supabase signed URL; Jamendo/Audius resolve theo adapter. Player chỉ biết endpoint này, không tự xử lý secret/provider logic.

**Audit bắt buộc trước khi rời phase.** Audit anon/pending bị chặn, approved được URL hợp lệ, removed/rejected trả status đúng.

**Khi audit FAIL.** Phase chưa hoàn thành. Codex phải ghi symptom, reproduction, expected, actual, root cause, file bị ảnh hưởng, cách sửa, regression test và residual risk vào báo cáo lỗi. Không được sang phase sau, không được tắt test, nới RLS, chuyển bucket public, thêm `any`/`@ts-ignore` hoặc catch lỗi giả thành công. Sau khi sửa phải chạy lại toàn bộ audit của phase này rồi mới đổi trạng thái sang PASS.


## M4.9 — Signed URL renewal contract

**Thực hiện.** Định nghĩa expiry/renew cho upload và provider nếu URL ngắn hạn. Client khi hết hạn xin URL mới và resume position thay vì restart từ đầu.

**Audit bắt buộc trước khi rời phase.** Audit giả lập URL hết hạn và resume gần vị trí cũ.

**Khi audit FAIL.** Phase chưa hoàn thành. Codex phải ghi symptom, reproduction, expected, actual, root cause, file bị ảnh hưởng, cách sửa, regression test và residual risk vào báo cáo lỗi. Không được sang phase sau, không được tắt test, nới RLS, chuyển bucket public, thêm `any`/`@ts-ignore` hoặc catch lỗi giả thành công. Sau khi sửa phải chạy lại toàn bộ audit của phase này rồi mới đổi trạng thái sang PASS.


## M4.10 — Trending API

**Thực hiện.** Tạo approved-only API gộp local DB, Audius, Jamendo bằng `Promise.allSettled`, giới hạn số item, normalize DTO và trả `partialSources` nếu một provider fail.

**Audit bắt buộc trước khi rời phase.** Audit tắt từng provider; route vẫn phục vụ nguồn còn lại mà không trả data giả.

**Khi audit FAIL.** Phase chưa hoàn thành. Codex phải ghi symptom, reproduction, expected, actual, root cause, file bị ảnh hưởng, cách sửa, regression test và residual risk vào báo cáo lỗi. Không được sang phase sau, không được tắt test, nới RLS, chuyển bucket public, thêm `any`/`@ts-ignore` hoặc catch lỗi giả thành công. Sau khi sửa phải chạy lại toàn bộ audit của phase này rồi mới đổi trạng thái sang PASS.


## M4.11 — Trending UI

**Thực hiện.** Trang chủ dùng TanStack Query, skeleton/error/empty state và source badge. Play action chỉ đưa canonical Track vào queue.

**Audit bắt buộc trước khi rời phase.** Audit UI khi loading, provider partial, empty và success.

**Khi audit FAIL.** Phase chưa hoàn thành. Codex phải ghi symptom, reproduction, expected, actual, root cause, file bị ảnh hưởng, cách sửa, regression test và residual risk vào báo cáo lỗi. Không được sang phase sau, không được tắt test, nới RLS, chuyển bucket public, thêm `any`/`@ts-ignore` hoặc catch lỗi giả thành công. Sau khi sửa phải chạy lại toàn bộ audit của phase này rồi mới đổi trạng thái sang PASS.


## M4.12 — Howler hook one-instance

**Thực hiện.** Tạo hook player giữ một Howl active, unload khi đổi bài và dùng html5 mode cho stream dài nếu phù hợp. Không new Howl mỗi render.

**Audit bắt buộc trước khi rời phase.** Audit đổi bài nhanh và đảm bảo không phát chồng.

**Khi audit FAIL.** Phase chưa hoàn thành. Codex phải ghi symptom, reproduction, expected, actual, root cause, file bị ảnh hưởng, cách sửa, regression test và residual risk vào báo cáo lỗi. Không được sang phase sau, không được tắt test, nới RLS, chuyển bucket public, thêm `any`/`@ts-ignore` hoặc catch lỗi giả thành công. Sau khi sửa phải chạy lại toàn bộ audit của phase này rồi mới đổi trạng thái sang PASS.


## M4.13 — PlayerBar play/pause

**Thực hiện.** Tạo PlayerBar duy nhất ở layout, điều khiển store/hook, hiển thị bài hiện tại và trạng thái playback.

**Audit bắt buộc trước khi rời phase.** Audit click nhanh play/pause và route navigation không tạo player thứ hai.

**Khi audit FAIL.** Phase chưa hoàn thành. Codex phải ghi symptom, reproduction, expected, actual, root cause, file bị ảnh hưởng, cách sửa, regression test và residual risk vào báo cáo lỗi. Không được sang phase sau, không được tắt test, nới RLS, chuyển bucket public, thêm `any`/`@ts-ignore` hoặc catch lỗi giả thành công. Sau khi sửa phải chạy lại toàn bộ audit của phase này rồi mới đổi trạng thái sang PASS.


## M4.14 — Seek

**Thực hiện.** Seek dùng giây thật, clamp 0-duration, UI timestamp chuẩn và không dựa vào phần trăm làm source-of-truth.

**Audit bắt buộc trước khi rời phase.** Audit seek 30s với sai số nhỏ và track duration ngắn.

**Khi audit FAIL.** Phase chưa hoàn thành. Codex phải ghi symptom, reproduction, expected, actual, root cause, file bị ảnh hưởng, cách sửa, regression test và residual risk vào báo cáo lỗi. Không được sang phase sau, không được tắt test, nới RLS, chuyển bucket public, thêm `any`/`@ts-ignore` hoặc catch lỗi giả thành công. Sau khi sửa phải chạy lại toàn bộ audit của phase này rồi mới đổi trạng thái sang PASS.


## M4.15 — Volume/mute

**Thực hiện.** Volume 0..1, mute restore volume hợp lý, persist preference qua store.

**Audit bắt buộc trước khi rời phase.** Audit reload và audio output thực tế.

**Khi audit FAIL.** Phase chưa hoàn thành. Codex phải ghi symptom, reproduction, expected, actual, root cause, file bị ảnh hưởng, cách sửa, regression test và residual risk vào báo cáo lỗi. Không được sang phase sau, không được tắt test, nới RLS, chuyển bucket public, thêm `any`/`@ts-ignore` hoặc catch lỗi giả thành công. Sau khi sửa phải chạy lại toàn bộ audit của phase này rồi mới đổi trạng thái sang PASS.


## M4.16 — Next/prev

**Thực hiện.** Implement boundary rõ: prev đầu queue, next cuối queue phụ thuộc repeat. Không để index âm/vượt length.

**Audit bắt buộc trước khi rời phase.** Audit queue 0,1,n bài.

**Khi audit FAIL.** Phase chưa hoàn thành. Codex phải ghi symptom, reproduction, expected, actual, root cause, file bị ảnh hưởng, cách sửa, regression test và residual risk vào báo cáo lỗi. Không được sang phase sau, không được tắt test, nới RLS, chuyển bucket public, thêm `any`/`@ts-ignore` hoặc catch lỗi giả thành công. Sau khi sửa phải chạy lại toàn bộ audit của phase này rồi mới đổi trạng thái sang PASS.


## M4.17 — Repeat

**Thực hiện.** Repeat off/one/all có state machine rõ. `ended` handler không được tự mâu thuẫn với next.

**Audit bắt buộc trước khi rời phase.** Audit từng mode qua track kết thúc giả lập/thật.

**Khi audit FAIL.** Phase chưa hoàn thành. Codex phải ghi symptom, reproduction, expected, actual, root cause, file bị ảnh hưởng, cách sửa, regression test và residual risk vào báo cáo lỗi. Không được sang phase sau, không được tắt test, nới RLS, chuyển bucket public, thêm `any`/`@ts-ignore` hoặc catch lỗi giả thành công. Sau khi sửa phải chạy lại toàn bộ audit của phase này rồi mới đổi trạng thái sang PASS.


## M4.18 — Shuffle

**Thực hiện.** Shuffle không làm mất/nhân đôi bài, giữ current track hợp lý và có thể tắt để trở về deterministic queue policy đã chọn.

**Audit bắt buộc trước khi rời phase.** Audit 100 lần shuffle trên fixture nhỏ bằng unit test.

**Khi audit FAIL.** Phase chưa hoàn thành. Codex phải ghi symptom, reproduction, expected, actual, root cause, file bị ảnh hưởng, cách sửa, regression test và residual risk vào báo cáo lỗi. Không được sang phase sau, không được tắt test, nới RLS, chuyển bucket public, thêm `any`/`@ts-ignore` hoặc catch lỗi giả thành công. Sau khi sửa phải chạy lại toàn bộ audit của phase này rồi mới đổi trạng thái sang PASS.


## M4.19 — Hotkeys

**Thực hiện.** Space, ArrowLeft/Right, M chỉ hoạt động ngoài input/textarea/contenteditable và không phá scroll/form.

**Audit bắt buộc trước khi rời phase.** Audit accessibility keyboard trên login/search/player.

**Khi audit FAIL.** Phase chưa hoàn thành. Codex phải ghi symptom, reproduction, expected, actual, root cause, file bị ảnh hưởng, cách sửa, regression test và residual risk vào báo cáo lỗi. Không được sang phase sau, không được tắt test, nới RLS, chuyển bucket public, thêm `any`/`@ts-ignore` hoặc catch lỗi giả thành công. Sau khi sửa phải chạy lại toàn bộ audit của phase này rồi mới đổi trạng thái sang PASS.


## M4.20 — Pre-resolve next track

**Thực hiện.** Chỉ resolve URL bài kế tiếp gần cuối bài; không tạo Howl thứ hai tự phát. Đây là optimization có thể disable nếu provider rate-limit.

**Audit bắt buộc trước khi rời phase.** Audit network không tải vô hạn queue.

**Khi audit FAIL.** Phase chưa hoàn thành. Codex phải ghi symptom, reproduction, expected, actual, root cause, file bị ảnh hưởng, cách sửa, regression test và residual risk vào báo cáo lỗi. Không được sang phase sau, không được tắt test, nới RLS, chuyển bucket public, thêm `any`/`@ts-ignore` hoặc catch lỗi giả thành công. Sau khi sửa phải chạy lại toàn bộ audit của phase này rồi mới đổi trạng thái sang PASS.


## M4.21 — History write >5s

**Thực hiện.** Player chỉ POST history sau ngưỡng nghe thực >5s và tối đa một record theo policy mỗi playback session. Không ghi ngay khi click Play.

**Audit bắt buộc trước khi rời phase.** Audit nghe 3s không row, 7s có row.

**Khi audit FAIL.** Phase chưa hoàn thành. Codex phải ghi symptom, reproduction, expected, actual, root cause, file bị ảnh hưởng, cách sửa, regression test và residual risk vào báo cáo lỗi. Không được sang phase sau, không được tắt test, nới RLS, chuyển bucket public, thêm `any`/`@ts-ignore` hoặc catch lỗi giả thành công. Sau khi sửa phải chạy lại toàn bộ audit của phase này rồi mới đổi trạng thái sang PASS.


## M4.22 — Provider health

**Thực hiện.** Admin health route kiểm tra adapter có timeout. Nếu dùng circuit breaker production thì lưu state Redis; nếu không cần, chỉ allSettled/timeout và status hiện tại, không giả vờ có circuit breaker in-memory bền vững.

**Audit bắt buộc trước khi rời phase.** Audit serverless restart không làm logic phụ thuộc RAM.

**Khi audit FAIL.** Phase chưa hoàn thành. Codex phải ghi symptom, reproduction, expected, actual, root cause, file bị ảnh hưởng, cách sửa, regression test và residual risk vào báo cáo lỗi. Không được sang phase sau, không được tắt test, nới RLS, chuyển bucket public, thêm `any`/`@ts-ignore` hoặc catch lỗi giả thành công. Sau khi sửa phải chạy lại toàn bộ audit của phase này rồi mới đổi trạng thái sang PASS.


## M4.23 — Player unit tests

**Thực hiện.** Vitest cho queue, repeat, shuffle, boundary và resolver helpers. Không mock toàn bộ logic đến mức test vô nghĩa.

**Audit bắt buộc trước khi rời phase.** Audit chạy nhiều lần deterministic.

**Khi audit FAIL.** Phase chưa hoàn thành. Codex phải ghi symptom, reproduction, expected, actual, root cause, file bị ảnh hưởng, cách sửa, regression test và residual risk vào báo cáo lỗi. Không được sang phase sau, không được tắt test, nới RLS, chuyển bucket public, thêm `any`/`@ts-ignore` hoặc catch lỗi giả thành công. Sau khi sửa phải chạy lại toàn bộ audit của phase này rồi mới đổi trạng thái sang PASS.


## M4.24 — Player/catalog gate

**Thực hiện.** Chạy playback thật với upload nội bộ và các provider ngoài đã vượt compatibility gate, kiểm tra seek, renew URL, queue reload, partial provider và lỗi mạng. Nếu một provider không đủ điều kiện thì phải ghi quyết định disable/fallback rõ, không giả vờ PASS.

**Audit bắt buộc trước khi rời phase.** Audit bằng evidence playback thực, không chỉ unit test. PASS khi player hoạt động đúng với mọi nguồn còn được bật trong MVP.

**Khi audit FAIL.** Phase chưa hoàn thành. Codex phải ghi symptom, reproduction, expected, actual, root cause, file bị ảnh hưởng, cách sửa, regression test và residual risk vào báo cáo lỗi. Không được sang phase sau, không được tắt test, nới RLS, chuyển bucket public, thêm `any`/`@ts-ignore` hoặc catch lỗi giả thành công. Sau khi sửa phải chạy lại toàn bộ audit của phase này rồi mới đổi trạng thái sang PASS.


# KHỐI Q5 — SOCIAL + SEARCH + ADMIN


## Q5.1 — Like API

**Thực hiện.** API chỉ gọi RPC toggle_like atomic; user lấy từ session/auth context, không nhận user_id từ body. Không dùng select-then-insert ở Node vì có race condition.

**Audit bắt buộc trước khi rời phase.** Audit hai request gần đồng thời và reload persistence; DB không có row trùng hoặc lỗi invariant.

**Khi audit FAIL.** Phase chưa hoàn thành. Codex phải ghi symptom, reproduction, expected, actual, root cause, file bị ảnh hưởng, cách sửa, regression test và residual risk vào báo cáo lỗi. Không được sang phase sau, không được tắt test, nới RLS, chuyển bucket public, thêm `any`/`@ts-ignore` hoặc catch lỗi giả thành công. Sau khi sửa phải chạy lại toàn bộ audit của phase này rồi mới đổi trạng thái sang PASS.


## Q5.2 — Like UI optimistic

**Thực hiện.** UI có optimistic update nhưng phải rollback khi API fail và invalidate/refetch sau success để server vẫn là nguồn thật.

**Audit bắt buộc trước khi rời phase.** Audit ép 500 và xác nhận UI quay về trạng thái server.

**Khi audit FAIL.** Phase chưa hoàn thành. Codex phải ghi symptom, reproduction, expected, actual, root cause, file bị ảnh hưởng, cách sửa, regression test và residual risk vào báo cáo lỗi. Không được sang phase sau, không được tắt test, nới RLS, chuyển bucket public, thêm `any`/`@ts-ignore` hoặc catch lỗi giả thành công. Sau khi sửa phải chạy lại toàn bộ audit của phase này rồi mới đổi trạng thái sang PASS.


## Q5.3 — Playlist CRUD API

**Thực hiện.** CRUD playlist dùng session, RLS và validation tên. Owner luôn lấy từ auth, không lấy từ JSON body.

**Audit bắt buộc trước khi rời phase.** Audit A/B cross-user với private/public playlist.

**Khi audit FAIL.** Phase chưa hoàn thành. Codex phải ghi symptom, reproduction, expected, actual, root cause, file bị ảnh hưởng, cách sửa, regression test và residual risk vào báo cáo lỗi. Không được sang phase sau, không được tắt test, nới RLS, chuyển bucket public, thêm `any`/`@ts-ignore` hoặc catch lỗi giả thành công. Sau khi sửa phải chạy lại toàn bộ audit của phase này rồi mới đổi trạng thái sang PASS.


## Q5.4 — Playlist UI

**Thực hiện.** Tạo library và detail page có loading, error, empty state rõ; không sao chép logic security từ server vào client như một lớp bảo vệ giả.

**Audit bắt buộc trước khi rời phase.** Audit reload và navigation, private playlist vẫn đúng.

**Khi audit FAIL.** Phase chưa hoàn thành. Codex phải ghi symptom, reproduction, expected, actual, root cause, file bị ảnh hưởng, cách sửa, regression test và residual risk vào báo cáo lỗi. Không được sang phase sau, không được tắt test, nới RLS, chuyển bucket public, thêm `any`/`@ts-ignore` hoặc catch lỗi giả thành công. Sau khi sửa phải chạy lại toàn bộ audit của phase này rồi mới đổi trạng thái sang PASS.


## Q5.5 — Append track atomic

**Thực hiện.** UI/API gọi RPC append_playlist_track atomic đã tạo ở D2.16. Không tính MAX(position)+1 ở client/server route.

**Audit bắt buộc trước khi rời phase.** Audit hai tab cùng thêm và duplicate track.

**Khi audit FAIL.** Phase chưa hoàn thành. Codex phải ghi symptom, reproduction, expected, actual, root cause, file bị ảnh hưởng, cách sửa, regression test và residual risk vào báo cáo lỗi. Không được sang phase sau, không được tắt test, nới RLS, chuyển bucket public, thêm `any`/`@ts-ignore` hoặc catch lỗi giả thành công. Sau khi sửa phải chạy lại toàn bộ audit của phase này rồi mới đổi trạng thái sang PASS.


## Q5.6 — Reorder atomic

**Thực hiện.** UI gọi RPC reorder và rollback optimistic khi lỗi. Không PATCH hai position rời rạc.

**Audit bắt buộc trước khi rời phase.** Audit double-click và hai tab; unique position vẫn đúng.

**Khi audit FAIL.** Phase chưa hoàn thành. Codex phải ghi symptom, reproduction, expected, actual, root cause, file bị ảnh hưởng, cách sửa, regression test và residual risk vào báo cáo lỗi. Không được sang phase sau, không được tắt test, nới RLS, chuyển bucket public, thêm `any`/`@ts-ignore` hoặc catch lỗi giả thành công. Sau khi sửa phải chạy lại toàn bộ audit của phase này rồi mới đổi trạng thái sang PASS.


## Q5.7 — History API/UI

**Thực hiện.** Hiển thị lịch sử của chính user theo thời gian, giới hạn/pagination hợp lý và cho xóa mục thuộc mình. Không expose history user khác.

**Audit bắt buộc trước khi rời phase.** Audit B không đọc/xóa history A.

**Khi audit FAIL.** Phase chưa hoàn thành. Codex phải ghi symptom, reproduction, expected, actual, root cause, file bị ảnh hưởng, cách sửa, regression test và residual risk vào báo cáo lỗi. Không được sang phase sau, không được tắt test, nới RLS, chuyển bucket public, thêm `any`/`@ts-ignore` hoặc catch lỗi giả thành công. Sau khi sửa phải chạy lại toàn bộ audit của phase này rồi mới đổi trạng thái sang PASS.


## Q5.8 — Search API contract

**Thực hiện.** GET search trim query, giới hạn độ dài, rate-limit và dùng allSettled cho local/Audius/Jamendo. Query rỗng trả lỗi nghiệp vụ; provider fail chỉ làm partialSources.

**Audit bắt buộc trước khi rời phase.** Audit unicode, input dài, ký tự đặc biệt và một provider timeout.

**Khi audit FAIL.** Phase chưa hoàn thành. Codex phải ghi symptom, reproduction, expected, actual, root cause, file bị ảnh hưởng, cách sửa, regression test và residual risk vào báo cáo lỗi. Không được sang phase sau, không được tắt test, nới RLS, chuyển bucket public, thêm `any`/`@ts-ignore` hoặc catch lỗi giả thành công. Sau khi sửa phải chạy lại toàn bộ audit của phase này rồi mới đổi trạng thái sang PASS.


## Q5.9 — Local FTS đúng index

**Thực hiện.** Search local phải dùng search_text/index đã thiết kế, FTS trước và trigram fallback; không viết biểu thức làm index vô dụng.

**Audit bắt buộc trước khi rời phase.** Audit EXPLAIN và query không dấu thực tế.

**Khi audit FAIL.** Phase chưa hoàn thành. Codex phải ghi symptom, reproduction, expected, actual, root cause, file bị ảnh hưởng, cách sửa, regression test và residual risk vào báo cáo lỗi. Không được sang phase sau, không được tắt test, nới RLS, chuyển bucket public, thêm `any`/`@ts-ignore` hoặc catch lỗi giả thành công. Sau khi sửa phải chạy lại toàn bộ audit của phase này rồi mới đổi trạng thái sang PASS.


## Q5.10 — Search ranking merge

**Thực hiện.** Chuẩn hóa score giữa các nguồn theo rule cố định, dedup source/id và limit cuối cùng. Không cho provider có scale lớn hơn lấn toàn bộ list chỉ do khác thang điểm.

**Audit bắt buộc trước khi rời phase.** Audit bằng fixture deterministic.

**Khi audit FAIL.** Phase chưa hoàn thành. Codex phải ghi symptom, reproduction, expected, actual, root cause, file bị ảnh hưởng, cách sửa, regression test và residual risk vào báo cáo lỗi. Không được sang phase sau, không được tắt test, nới RLS, chuyển bucket public, thêm `any`/`@ts-ignore` hoặc catch lỗi giả thành công. Sau khi sửa phải chạy lại toàn bộ audit của phase này rồi mới đổi trạng thái sang PASS.


## Q5.11 — Search UI highlight an toàn

**Thực hiện.** Highlight bằng React nodes/text escaping, không dùng dangerouslySetInnerHTML với title/artist từ provider.

**Audit bắt buộc trước khi rời phase.** Audit payload chứa HTML/script chỉ hiển thị text.

**Khi audit FAIL.** Phase chưa hoàn thành. Codex phải ghi symptom, reproduction, expected, actual, root cause, file bị ảnh hưởng, cách sửa, regression test và residual risk vào báo cáo lỗi. Không được sang phase sau, không được tắt test, nới RLS, chuyển bucket public, thêm `any`/`@ts-ignore` hoặc catch lỗi giả thành công. Sau khi sửa phải chạy lại toàn bộ audit của phase này rồi mới đổi trạng thái sang PASS.


## Q5.12 — Reports API

**Thực hiện.** Approved user được report một track một lần, reason validate server-side, duplicate trả conflict hợp lý.

**Audit bắt buộc trước khi rời phase.** Audit duplicate, anonymous và pending user.

**Khi audit FAIL.** Phase chưa hoàn thành. Codex phải ghi symptom, reproduction, expected, actual, root cause, file bị ảnh hưởng, cách sửa, regression test và residual risk vào báo cáo lỗi. Không được sang phase sau, không được tắt test, nới RLS, chuyển bucket public, thêm `any`/`@ts-ignore` hoặc catch lỗi giả thành công. Sau khi sửa phải chạy lại toàn bộ audit của phase này rồi mới đổi trạng thái sang PASS.


## Q5.13 — Admin page

**Thực hiện.** Admin page phải gate ở server trước khi render dữ liệu nhạy cảm; client button chỉ là UX, không phải security boundary.

**Audit bắt buộc trước khi rời phase.** Audit user thường truy cập URL trực tiếp và gọi API trực tiếp.

**Khi audit FAIL.** Phase chưa hoàn thành. Codex phải ghi symptom, reproduction, expected, actual, root cause, file bị ảnh hưởng, cách sửa, regression test và residual risk vào báo cáo lỗi. Không được sang phase sau, không được tắt test, nới RLS, chuyển bucket public, thêm `any`/`@ts-ignore` hoặc catch lỗi giả thành công. Sau khi sửa phải chạy lại toàn bộ audit của phase này rồi mới đổi trạng thái sang PASS.


## Q5.14 — Moderation transaction

**Thực hiện.** Duyệt/reject/remove track và audit_log phải cùng transaction/RPC; không ghi success trước mutation.

**Audit bắt buộc trước khi rời phase.** Audit ép DB fail và xác nhận không có success log giả.

**Khi audit FAIL.** Phase chưa hoàn thành. Codex phải ghi symptom, reproduction, expected, actual, root cause, file bị ảnh hưởng, cách sửa, regression test và residual risk vào báo cáo lỗi. Không được sang phase sau, không được tắt test, nới RLS, chuyển bucket public, thêm `any`/`@ts-ignore` hoặc catch lỗi giả thành công. Sau khi sửa phải chạy lại toàn bộ audit của phase này rồi mới đổi trạng thái sang PASS.


## Q5.15 — Provider health UI

**Thực hiện.** Admin xem status provider/timestamp. Không tự mark removed hàng loạt chỉ vì timeout tạm thời.

**Audit bắt buộc trước khi rời phase.** Audit provider fail ngắn hạn không phá catalog.

**Khi audit FAIL.** Phase chưa hoàn thành. Codex phải ghi symptom, reproduction, expected, actual, root cause, file bị ảnh hưởng, cách sửa, regression test và residual risk vào báo cáo lỗi. Không được sang phase sau, không được tắt test, nới RLS, chuyển bucket public, thêm `any`/`@ts-ignore` hoặc catch lỗi giả thành công. Sau khi sửa phải chạy lại toàn bộ audit của phase này rồi mới đổi trạng thái sang PASS.


## Q5.16 — Quota settings

**Thực hiện.** Trang settings hiển thị usage do server/DB tính theo rule used+reservation, không tự cộng các item đang render.

**Audit bắt buộc trước khi rời phase.** Audit số liệu so với SQL/RPC.

**Khi audit FAIL.** Phase chưa hoàn thành. Codex phải ghi symptom, reproduction, expected, actual, root cause, file bị ảnh hưởng, cách sửa, regression test và residual risk vào báo cáo lỗi. Không được sang phase sau, không được tắt test, nới RLS, chuyển bucket public, thêm `any`/`@ts-ignore` hoặc catch lỗi giả thành công. Sau khi sửa phải chạy lại toàn bộ audit của phase này rồi mới đổi trạng thái sang PASS.


## Q5.17 — Upload UI

**Thực hiện.** Flow sign → PUT → complete có progress, cancel/error rõ, consent bắt buộc và retry idempotent.

**Audit bắt buộc trước khi rời phase.** Audit mạng đứt giữa upload và complete rồi maintenance có thể dọn rác.

**Khi audit FAIL.** Phase chưa hoàn thành. Codex phải ghi symptom, reproduction, expected, actual, root cause, file bị ảnh hưởng, cách sửa, regression test và residual risk vào báo cáo lỗi. Không được sang phase sau, không được tắt test, nới RLS, chuyển bucket public, thêm `any`/`@ts-ignore` hoặc catch lỗi giả thành công. Sau khi sửa phải chạy lại toàn bộ audit của phase này rồi mới đổi trạng thái sang PASS.


## Q5.18 — Upload abuse integration

**Thực hiện.** Test traversal, fake MIME, oversize, quota race, duplicate hash, complete replay và user B dùng reservation A.

**Audit bắt buộc trước khi rời phase.** Audit tất cả deny case bị chặn bởi server/DB thực.

**Khi audit FAIL.** Phase chưa hoàn thành. Codex phải ghi symptom, reproduction, expected, actual, root cause, file bị ảnh hưởng, cách sửa, regression test và residual risk vào báo cáo lỗi. Không được sang phase sau, không được tắt test, nới RLS, chuyển bucket public, thêm `any`/`@ts-ignore` hoặc catch lỗi giả thành công. Sau khi sửa phải chạy lại toàn bộ audit của phase này rồi mới đổi trạng thái sang PASS.


## Q5.19 — Terms/privacy

**Thực hiện.** Trang terms/privacy nói rõ scope private/học tập, quyền nội dung, xóa account và private không đồng nghĩa miễn bản quyền.

**Audit bắt buộc trước khi rời phase.** Audit link từ login/footer và không có tuyên bố pháp lý sai.

**Khi audit FAIL.** Phase chưa hoàn thành. Codex phải ghi symptom, reproduction, expected, actual, root cause, file bị ảnh hưởng, cách sửa, regression test và residual risk vào báo cáo lỗi. Không được sang phase sau, không được tắt test, nới RLS, chuyển bucket public, thêm `any`/`@ts-ignore` hoặc catch lỗi giả thành công. Sau khi sửa phải chạy lại toàn bộ audit của phase này rồi mới đổi trạng thái sang PASS.


## Q5.20 — API docs

**Thực hiện.** Tạo `/docs/api` với route, auth requirement, request/response mẫu và error semantics; tuyệt đối không ghi secret thật.

**Audit bắt buộc trước khi rời phase.** Audit curl mẫu local.

**Khi audit FAIL.** Phase chưa hoàn thành. Codex phải ghi symptom, reproduction, expected, actual, root cause, file bị ảnh hưởng, cách sửa, regression test và residual risk vào báo cáo lỗi. Không được sang phase sau, không được tắt test, nới RLS, chuyển bucket public, thêm `any`/`@ts-ignore` hoặc catch lỗi giả thành công. Sau khi sửa phải chạy lại toàn bộ audit của phase này rồi mới đổi trạng thái sang PASS.


## Q5.21 — README developer

**Thực hiện.** README mô tả migration-only local-first: supabase start/reset/test, env, npm commands, deploy và nguyên tắc không sửa cloud DB trực tiếp.

**Audit bắt buộc trước khi rời phase.** Audit clone/worktree sạch có thể làm theo.

**Khi audit FAIL.** Phase chưa hoàn thành. Codex phải ghi symptom, reproduction, expected, actual, root cause, file bị ảnh hưởng, cách sửa, regression test và residual risk vào báo cáo lỗi. Không được sang phase sau, không được tắt test, nới RLS, chuyển bucket public, thêm `any`/`@ts-ignore` hoặc catch lỗi giả thành công. Sau khi sửa phải chạy lại toàn bộ audit của phase này rồi mới đổi trạng thái sang PASS.


## Q5.22 — Dead-code audit

**Thực hiện.** Tìm implementation trùng sau refactor: Supabase client, logger, Track type, player store, provider adapter, old API route. Xóa chỉ sau khi xác nhận không còn import.

**Audit bắt buộc trước khi rời phase.** Audit build và tests sau cleanup.

**Khi audit FAIL.** Phase chưa hoàn thành. Codex phải ghi symptom, reproduction, expected, actual, root cause, file bị ảnh hưởng, cách sửa, regression test và residual risk vào báo cáo lỗi. Không được sang phase sau, không được tắt test, nới RLS, chuyển bucket public, thêm `any`/`@ts-ignore` hoặc catch lỗi giả thành công. Sau khi sửa phải chạy lại toàn bộ audit của phase này rồi mới đổi trạng thái sang PASS.


## Q5.23 — Dependency audit

**Thực hiện.** Kiểm tra package.json không còn dependency thử nghiệm, ORM ngoài plan hoặc thư viện chức năng trùng.

**Audit bắt buộc trước khi rời phase.** Audit npm ls, import grep và build.

**Khi audit FAIL.** Phase chưa hoàn thành. Codex phải ghi symptom, reproduction, expected, actual, root cause, file bị ảnh hưởng, cách sửa, regression test và residual risk vào báo cáo lỗi. Không được sang phase sau, không được tắt test, nới RLS, chuyển bucket public, thêm `any`/`@ts-ignore` hoặc catch lỗi giả thành công. Sau khi sửa phải chạy lại toàn bộ audit của phase này rồi mới đổi trạng thái sang PASS.


## Q5.24 — Feature gate social/search/admin

**Thực hiện.** Chạy like, playlist, history, search, upload, moderation và settings bằng role thật trên local.

**Audit bắt buộc trước khi rời phase.** Audit RLS negative suite vẫn xanh sau toàn bộ feature.

**Khi audit FAIL.** Phase chưa hoàn thành. Codex phải ghi symptom, reproduction, expected, actual, root cause, file bị ảnh hưởng, cách sửa, regression test và residual risk vào báo cáo lỗi. Không được sang phase sau, không được tắt test, nới RLS, chuyển bucket public, thêm `any`/`@ts-ignore` hoặc catch lỗi giả thành công. Sau khi sửa phải chạy lại toàn bộ audit của phase này rồi mới đổi trạng thái sang PASS.


# KHỐI R6 — TEST + VẬN HÀNH + DEPLOY


## R6.1 — Vitest complete

**Thực hiện.** Unit test helpers, queue, ranking, quota boundary và logic thuần quan trọng; đưa vào CI.

**Audit bắt buộc trước khi rời phase.** Audit chạy nhiều lần deterministic.

**Khi audit FAIL.** Phase chưa hoàn thành. Codex phải ghi symptom, reproduction, expected, actual, root cause, file bị ảnh hưởng, cách sửa, regression test và residual risk vào báo cáo lỗi. Không được sang phase sau, không được tắt test, nới RLS, chuyển bucket public, thêm `any`/`@ts-ignore` hoặc catch lỗi giả thành công. Sau khi sửa phải chạy lại toàn bộ audit của phase này rồi mới đổi trạng thái sang PASS.


## R6.2 — Playwright deterministic setup

**Thực hiện.** Cấu hình Chromium, local webServer, account/fixture test và mock provider ngoài khi test mục tiêu UI. Không dùng production account.

**Audit bắt buộc trước khi rời phase.** Audit `npx playwright test --list` và fixture reset.

**Khi audit FAIL.** Phase chưa hoàn thành. Codex phải ghi symptom, reproduction, expected, actual, root cause, file bị ảnh hưởng, cách sửa, regression test và residual risk vào báo cáo lỗi. Không được sang phase sau, không được tắt test, nới RLS, chuyển bucket public, thêm `any`/`@ts-ignore` hoặc catch lỗi giả thành công. Sau khi sửa phải chạy lại toàn bộ audit của phase này rồi mới đổi trạng thái sang PASS.


## R6.3 — E2E Auth

**Thực hiện.** Test approved login, pending denial, reload session và logout.

**Audit bắt buộc trước khi rời phase.** Audit selector ổn định, không sleep cứng.

**Khi audit FAIL.** Phase chưa hoàn thành. Codex phải ghi symptom, reproduction, expected, actual, root cause, file bị ảnh hưởng, cách sửa, regression test và residual risk vào báo cáo lỗi. Không được sang phase sau, không được tắt test, nới RLS, chuyển bucket public, thêm `any`/`@ts-ignore` hoặc catch lỗi giả thành công. Sau khi sửa phải chạy lại toàn bộ audit của phase này rồi mới đổi trạng thái sang PASS.


## R6.4 — E2E Listen

**Thực hiện.** Test login → fixture track → play → elapsed tăng → seek → pause bằng audio fixture có quyền.

**Audit bắt buộc trước khi rời phase.** Audit ba lần liên tiếp xanh.

**Khi audit FAIL.** Phase chưa hoàn thành. Codex phải ghi symptom, reproduction, expected, actual, root cause, file bị ảnh hưởng, cách sửa, regression test và residual risk vào báo cáo lỗi. Không được sang phase sau, không được tắt test, nới RLS, chuyển bucket public, thêm `any`/`@ts-ignore` hoặc catch lỗi giả thành công. Sau khi sửa phải chạy lại toàn bộ audit của phase này rồi mới đổi trạng thái sang PASS.


## R6.5 — E2E Search

**Thực hiện.** Test query không dấu ra local fixture có dấu và highlight không inject HTML.

**Audit bắt buộc trước khi rời phase.** Audit ba lần liên tiếp xanh.

**Khi audit FAIL.** Phase chưa hoàn thành. Codex phải ghi symptom, reproduction, expected, actual, root cause, file bị ảnh hưởng, cách sửa, regression test và residual risk vào báo cáo lỗi. Không được sang phase sau, không được tắt test, nới RLS, chuyển bucket public, thêm `any`/`@ts-ignore` hoặc catch lỗi giả thành công. Sau khi sửa phải chạy lại toàn bộ audit của phase này rồi mới đổi trạng thái sang PASS.


## R6.6 — E2E Social

**Thực hiện.** Test like reload, playlist create/add/reorder reload và user B không thấy private playlist.

**Audit bắt buộc trước khi rời phase.** Audit data cleanup/reset deterministic.

**Khi audit FAIL.** Phase chưa hoàn thành. Codex phải ghi symptom, reproduction, expected, actual, root cause, file bị ảnh hưởng, cách sửa, regression test và residual risk vào báo cáo lỗi. Không được sang phase sau, không được tắt test, nới RLS, chuyển bucket public, thêm `any`/`@ts-ignore` hoặc catch lỗi giả thành công. Sau khi sửa phải chạy lại toàn bộ audit của phase này rồi mới đổi trạng thái sang PASS.


## R6.7 — E2E Upload

**Thực hiện.** Test file hợp lệ lên pending, fake MIME fail và duplicate fail.

**Audit bắt buộc trước khi rời phase.** Audit không còn object/reservation rác ngoài policy cleanup.

**Khi audit FAIL.** Phase chưa hoàn thành. Codex phải ghi symptom, reproduction, expected, actual, root cause, file bị ảnh hưởng, cách sửa, regression test và residual risk vào báo cáo lỗi. Không được sang phase sau, không được tắt test, nới RLS, chuyển bucket public, thêm `any`/`@ts-ignore` hoặc catch lỗi giả thành công. Sau khi sửa phải chạy lại toàn bộ audit của phase này rồi mới đổi trạng thái sang PASS.


## R6.8 — Security regression suite

**Thực hiện.** Chạy pgTAP, API abuse, cross-user, role escalation và bundle secret grep trong một gate.

**Audit bắt buộc trước khi rời phase.** Audit bất kỳ deny test nào thành allow là blocking.

**Khi audit FAIL.** Phase chưa hoàn thành. Codex phải ghi symptom, reproduction, expected, actual, root cause, file bị ảnh hưởng, cách sửa, regression test và residual risk vào báo cáo lỗi. Không được sang phase sau, không được tắt test, nới RLS, chuyển bucket public, thêm `any`/`@ts-ignore` hoặc catch lỗi giả thành công. Sau khi sửa phải chạy lại toàn bộ audit của phase này rồi mới đổi trạng thái sang PASS.


## R6.9 — Health endpoint

**Thực hiện.** Tạo `/api/health` no-store, timeout ngắn, trả 503 khi dependency chính fail và không lộ env.

**Audit bắt buộc trước khi rời phase.** Audit DB up/down.

**Khi audit FAIL.** Phase chưa hoàn thành. Codex phải ghi symptom, reproduction, expected, actual, root cause, file bị ảnh hưởng, cách sửa, regression test và residual risk vào báo cáo lỗi. Không được sang phase sau, không được tắt test, nới RLS, chuyển bucket public, thêm `any`/`@ts-ignore` hoặc catch lỗi giả thành công. Sau khi sửa phải chạy lại toàn bộ audit của phase này rồi mới đổi trạng thái sang PASS.


## R6.10 — Maintenance route

**Thực hiện.** Tạo route maintenance xác thực CRON_SECRET, idempotent, dọn history/reservation/object rác theo runbook.

**Audit bắt buộc trước khi rời phase.** Audit unauthorized 401 và authorized chạy hai lần an toàn.

**Khi audit FAIL.** Phase chưa hoàn thành. Codex phải ghi symptom, reproduction, expected, actual, root cause, file bị ảnh hưởng, cách sửa, regression test và residual risk vào báo cáo lỗi. Không được sang phase sau, không được tắt test, nới RLS, chuyển bucket public, thêm `any`/`@ts-ignore` hoặc catch lỗi giả thành công. Sau khi sửa phải chạy lại toàn bộ audit của phase này rồi mới đổi trạng thái sang PASS.


## R6.11 — App error boundaries

**Thực hiện.** Tạo App Router error/global-error và nối requestId cho lỗi API.

**Audit bắt buộc trước khi rời phase.** Audit ép lỗi render/API.

**Khi audit FAIL.** Phase chưa hoàn thành. Codex phải ghi symptom, reproduction, expected, actual, root cause, file bị ảnh hưởng, cách sửa, regression test và residual risk vào báo cáo lỗi. Không được sang phase sau, không được tắt test, nới RLS, chuyển bucket public, thêm `any`/`@ts-ignore` hoặc catch lỗi giả thành công. Sau khi sửa phải chạy lại toàn bộ audit của phase này rồi mới đổi trạng thái sang PASS.


## R6.12 — Rate-limit abstraction

**Thực hiện.** Local có thể dùng in-memory adapter chỉ dev; production dùng shared Redis/Upstash. Key ưu tiên user-id, IP chỉ fallback từ nguồn platform tin cậy.

**Audit bắt buộc trước khi rời phase.** Audit spam vượt ngưỡng và reset window.

**Khi audit FAIL.** Phase chưa hoàn thành. Codex phải ghi symptom, reproduction, expected, actual, root cause, file bị ảnh hưởng, cách sửa, regression test và residual risk vào báo cáo lỗi. Không được sang phase sau, không được tắt test, nới RLS, chuyển bucket public, thêm `any`/`@ts-ignore` hoặc catch lỗi giả thành công. Sau khi sửa phải chạy lại toàn bộ audit của phase này rồi mới đổi trạng thái sang PASS.


## R6.13 — Backup runbook

**Thực hiện.** RUNBOOK mô tả DB dump, migration history, Storage manifest/bytes và restore vào project test; không giả định SQL dump chứa audio file.

**Audit bắt buộc trước khi rời phase.** Audit restore drill metadata + ít nhất một object.

**Khi audit FAIL.** Phase chưa hoàn thành. Codex phải ghi symptom, reproduction, expected, actual, root cause, file bị ảnh hưởng, cách sửa, regression test và residual risk vào báo cáo lỗi. Không được sang phase sau, không được tắt test, nới RLS, chuyển bucket public, thêm `any`/`@ts-ignore` hoặc catch lỗi giả thành công. Sau khi sửa phải chạy lại toàn bộ audit của phase này rồi mới đổi trạng thái sang PASS.


## R6.14 — CI mở rộng

**Thực hiện.** CI chạy npm ci, tsc, lint, unit, build và database tests nếu runner có Supabase Local; E2E tách job phù hợp.

**Audit bắt buộc trước khi rời phase.** Audit remote CI từ commit sạch.

**Khi audit FAIL.** Phase chưa hoàn thành. Codex phải ghi symptom, reproduction, expected, actual, root cause, file bị ảnh hưởng, cách sửa, regression test và residual risk vào báo cáo lỗi. Không được sang phase sau, không được tắt test, nới RLS, chuyển bucket public, thêm `any`/`@ts-ignore` hoặc catch lỗi giả thành công. Sau khi sửa phải chạy lại toàn bộ audit của phase này rồi mới đổi trạng thái sang PASS.


## R6.15 — Predeploy migration audit

**Thực hiện.** Chạy migration list, db reset, db test, generate types và db push --dry-run trước cloud.

**Audit bắt buộc trước khi rời phase.** Audit dry-run chỉ có migration dự kiến và project link đúng.

**Khi audit FAIL.** Phase chưa hoàn thành. Codex phải ghi symptom, reproduction, expected, actual, root cause, file bị ảnh hưởng, cách sửa, regression test và residual risk vào báo cáo lỗi. Không được sang phase sau, không được tắt test, nới RLS, chuyển bucket public, thêm `any`/`@ts-ignore` hoặc catch lỗi giả thành công. Sau khi sửa phải chạy lại toàn bộ audit của phase này rồi mới đổi trạng thái sang PASS.


## R6.16 — Create/link Supabase Cloud

**Thực hiện.** Tạo/link production project, cấu hình env qua secret manager. Tuyệt đối không dùng db reset --linked production.

**Audit bắt buộc trước khi rời phase.** Audit project-ref và migration history.

**Khi audit FAIL.** Phase chưa hoàn thành. Codex phải ghi symptom, reproduction, expected, actual, root cause, file bị ảnh hưởng, cách sửa, regression test và residual risk vào báo cáo lỗi. Không được sang phase sau, không được tắt test, nới RLS, chuyển bucket public, thêm `any`/`@ts-ignore` hoặc catch lỗi giả thành công. Sau khi sửa phải chạy lại toàn bộ audit của phase này rồi mới đổi trạng thái sang PASS.


## R6.17 — Push production DB

**Thực hiện.** Chạy `npx supabase db push` chỉ sau R6.15 PASS, không include seed production.

**Audit bắt buộc trước khi rời phase.** Audit cloud schema hoạt động mà không cần SQL Editor vá tay.

**Khi audit FAIL.** Phase chưa hoàn thành. Codex phải ghi symptom, reproduction, expected, actual, root cause, file bị ảnh hưởng, cách sửa, regression test và residual risk vào báo cáo lỗi. Không được sang phase sau, không được tắt test, nới RLS, chuyển bucket public, thêm `any`/`@ts-ignore` hoặc catch lỗi giả thành công. Sau khi sửa phải chạy lại toàn bộ audit của phase này rồi mới đổi trạng thái sang PASS.


## R6.18 — Production Auth

**Thực hiện.** Tắt open signup, cấu hình redirect/invite. Google OAuth chỉ bật khi callback allowlist + approval gate hoàn chỉnh.

**Audit bắt buộc trước khi rời phase.** Audit anon/pending/approved.

**Khi audit FAIL.** Phase chưa hoàn thành. Codex phải ghi symptom, reproduction, expected, actual, root cause, file bị ảnh hưởng, cách sửa, regression test và residual risk vào báo cáo lỗi. Không được sang phase sau, không được tắt test, nới RLS, chuyển bucket public, thêm `any`/`@ts-ignore` hoặc catch lỗi giả thành công. Sau khi sửa phải chạy lại toàn bộ audit của phase này rồi mới đổi trạng thái sang PASS.


## R6.19 — Production Storage

**Thực hiện.** Xác nhận buckets private, policies và signed upload/download giống local.

**Audit bắt buộc trước khi rời phase.** Audit A/B/anon.

**Khi audit FAIL.** Phase chưa hoàn thành. Codex phải ghi symptom, reproduction, expected, actual, root cause, file bị ảnh hưởng, cách sửa, regression test và residual risk vào báo cáo lỗi. Không được sang phase sau, không được tắt test, nới RLS, chuyển bucket public, thêm `any`/`@ts-ignore` hoặc catch lỗi giả thành công. Sau khi sửa phải chạy lại toàn bộ audit của phase này rồi mới đổi trạng thái sang PASS.


## R6.20 — Vercel env

**Thực hiện.** Nhập đúng env production, phân biệt public/server-only, không copy local service credentials.

**Audit bắt buộc trước khi rời phase.** Audit build log/client không lộ secret.

**Khi audit FAIL.** Phase chưa hoàn thành. Codex phải ghi symptom, reproduction, expected, actual, root cause, file bị ảnh hưởng, cách sửa, regression test và residual risk vào báo cáo lỗi. Không được sang phase sau, không được tắt test, nới RLS, chuyển bucket public, thêm `any`/`@ts-ignore` hoặc catch lỗi giả thành công. Sau khi sửa phải chạy lại toàn bộ audit của phase này rồi mới đổi trạng thái sang PASS.


## R6.21 — Deploy Vercel

**Thực hiện.** Deploy khi full gate xanh; anonymous chỉ login/terms, approved nghe được, admin gate đúng.

**Audit bắt buộc trước khi rời phase.** Audit từ browser ẩn danh và mạng ngoài máy local.

**Khi audit FAIL.** Phase chưa hoàn thành. Codex phải ghi symptom, reproduction, expected, actual, root cause, file bị ảnh hưởng, cách sửa, regression test và residual risk vào báo cáo lỗi. Không được sang phase sau, không được tắt test, nới RLS, chuyển bucket public, thêm `any`/`@ts-ignore` hoặc catch lỗi giả thành công. Sau khi sửa phải chạy lại toàn bộ audit của phase này rồi mới đổi trạng thái sang PASS.


## R6.22 — Cron daily

**Thực hiện.** Bật một maintenance cron hằng ngày phù hợp giới hạn plan Vercel tại thời điểm deploy; không hard-code lịch nhiều lần/ngày cho Hobby.

**Audit bắt buộc trước khi rời phase.** Audit trigger tay và scheduled evidence; platform rule thay đổi thì update plan trước.

**Khi audit FAIL.** Phase chưa hoàn thành. Codex phải ghi symptom, reproduction, expected, actual, root cause, file bị ảnh hưởng, cách sửa, regression test và residual risk vào báo cáo lỗi. Không được sang phase sau, không được tắt test, nới RLS, chuyển bucket public, thêm `any`/`@ts-ignore` hoặc catch lỗi giả thành công. Sau khi sửa phải chạy lại toàn bộ audit của phase này rồi mới đổi trạng thái sang PASS.


## R6.23 — Performance budget

**Thực hiện.** Đo home/search/library/login trong điều kiện ghi rõ; mục tiêu demo <3s/trang nhưng không bịa số.

**Audit bắt buộc trước khi rời phase.** Audit lưu kết quả đo.

**Khi audit FAIL.** Phase chưa hoàn thành. Codex phải ghi symptom, reproduction, expected, actual, root cause, file bị ảnh hưởng, cách sửa, regression test và residual risk vào báo cáo lỗi. Không được sang phase sau, không được tắt test, nới RLS, chuyển bucket public, thêm `any`/`@ts-ignore` hoặc catch lỗi giả thành công. Sau khi sửa phải chạy lại toàn bộ audit của phase này rồi mới đổi trạng thái sang PASS.


## R6.24 — Production smoke matrix

**Thực hiện.** Chạy login, play/seek, search, like, playlist, upload pending, moderation, history và logout trên production test account.

**Audit bắt buộc trước khi rời phase.** Audit lỗi phải rollback/fix có kiểm soát, không vá DB trực tiếp.

**Khi audit FAIL.** Phase chưa hoàn thành. Codex phải ghi symptom, reproduction, expected, actual, root cause, file bị ảnh hưởng, cách sửa, regression test và residual risk vào báo cáo lỗi. Không được sang phase sau, không được tắt test, nới RLS, chuyển bucket public, thêm `any`/`@ts-ignore` hoặc catch lỗi giả thành công. Sau khi sửa phải chạy lại toàn bộ audit của phase này rồi mới đổi trạng thái sang PASS.


## R6.25 — Production secret leak scan

**Thực hiện.** Kiểm client bundle/source map/network cho service role, Audius bearer token và cron secret.

**Audit bắt buộc trước khi rời phase.** Audit zero secret leak trước release.

**Khi audit FAIL.** Phase chưa hoàn thành. Codex phải ghi symptom, reproduction, expected, actual, root cause, file bị ảnh hưởng, cách sửa, regression test và residual risk vào báo cáo lỗi. Không được sang phase sau, không được tắt test, nới RLS, chuyển bucket public, thêm `any`/`@ts-ignore` hoặc catch lỗi giả thành công. Sau khi sửa phải chạy lại toàn bộ audit của phase này rồi mới đổi trạng thái sang PASS.


## R6.26 — Rollback drill

**Thực hiện.** Thực hành rollback Vercel và forward-fix migration; không destructive reset production.

**Audit bắt buộc trước khi rời phase.** Audit runbook đủ rõ để agent khác làm.

**Khi audit FAIL.** Phase chưa hoàn thành. Codex phải ghi symptom, reproduction, expected, actual, root cause, file bị ảnh hưởng, cách sửa, regression test và residual risk vào báo cáo lỗi. Không được sang phase sau, không được tắt test, nới RLS, chuyển bucket public, thêm `any`/`@ts-ignore` hoặc catch lỗi giả thành công. Sau khi sửa phải chạy lại toàn bộ audit của phase này rồi mới đổi trạng thái sang PASS.


## R6.27 — Observability scope

**Thực hiện.** Structured logs/requestId là baseline; nếu chưa Sentry thì ghi accepted risk và trigger nâng cấp.

**Audit bắt buộc trước khi rời phase.** Audit tài liệu không tuyên bố monitoring hơn thực tế.

**Khi audit FAIL.** Phase chưa hoàn thành. Codex phải ghi symptom, reproduction, expected, actual, root cause, file bị ảnh hưởng, cách sửa, regression test và residual risk vào báo cáo lỗi. Không được sang phase sau, không được tắt test, nới RLS, chuyển bucket public, thêm `any`/`@ts-ignore` hoặc catch lỗi giả thành công. Sau khi sửa phải chạy lại toàn bộ audit của phase này rồi mới đổi trạng thái sang PASS.


## R6.28 — Operations gate

**Thực hiện.** Health, maintenance, backup/restore, rollback drill và production smoke đều phải PASS.

**Audit bắt buộc trước khi rời phase.** Audit evidence tập trung trong PHASES.

**Khi audit FAIL.** Phase chưa hoàn thành. Codex phải ghi symptom, reproduction, expected, actual, root cause, file bị ảnh hưởng, cách sửa, regression test và residual risk vào báo cáo lỗi. Không được sang phase sau, không được tắt test, nới RLS, chuyển bucket public, thêm `any`/`@ts-ignore` hoặc catch lỗi giả thành công. Sau khi sửa phải chạy lại toàn bộ audit của phase này rồi mới đổi trạng thái sang PASS.


# KHỐI Z7 — FINAL AUDIT + RELEASE LOCK


## Z7.1 — TODO/FIXME audit

**Thực hiện.** Quét TODO/FIXME/SKIP/deferred; mỗi mục phải giải quyết hoặc thành accepted risk/issue có owner và ngưỡng.

**Audit bắt buộc trước khi rời phase.** Audit rg + DECISIONS, cấm xóa comment chỉ để đẹp số.

**Khi audit FAIL.** Phase chưa hoàn thành. Codex phải ghi symptom, reproduction, expected, actual, root cause, file bị ảnh hưởng, cách sửa, regression test và residual risk vào báo cáo lỗi. Không được sang phase sau, không được tắt test, nới RLS, chuyển bucket public, thêm `any`/`@ts-ignore` hoặc catch lỗi giả thành công. Sau khi sửa phải chạy lại toàn bộ audit của phase này rồi mới đổi trạng thái sang PASS.


## Z7.2 — Clean-room local rebuild

**Thực hiện.** Từ worktree/clone sạch chạy npm ci, supabase start/reset/test, generate types, unit, lint, typecheck, build và E2E.

**Audit bắt buộc trước khi rời phase.** Audit toàn chuỗi xanh trong môi trường sạch.

**Khi audit FAIL.** Phase chưa hoàn thành. Codex phải ghi symptom, reproduction, expected, actual, root cause, file bị ảnh hưởng, cách sửa, regression test và residual risk vào báo cáo lỗi. Không được sang phase sau, không được tắt test, nới RLS, chuyển bucket public, thêm `any`/`@ts-ignore` hoặc catch lỗi giả thành công. Sau khi sửa phải chạy lại toàn bộ audit của phase này rồi mới đổi trạng thái sang PASS.


## Z7.3 — Security final

**Thực hiện.** Chạy lại A/B/C/admin/anon, escalation, playlist, Storage, upload abuse, secret scan và admin API.

**Audit bắt buộc trước khi rời phase.** Audit zero unexpected allow.

**Khi audit FAIL.** Phase chưa hoàn thành. Codex phải ghi symptom, reproduction, expected, actual, root cause, file bị ảnh hưởng, cách sửa, regression test và residual risk vào báo cáo lỗi. Không được sang phase sau, không được tắt test, nới RLS, chuyển bucket public, thêm `any`/`@ts-ignore` hoặc catch lỗi giả thành công. Sau khi sửa phải chạy lại toàn bộ audit của phase này rồi mới đổi trạng thái sang PASS.


## Z7.4 — Rubric 10/10 evidence

**Thực hiện.** Chấm build, audio, seek/volume, session, upload pending, search không dấu, like/playlist reload, RLS hai user, docs và performance bằng evidence thật.

**Audit bắt buộc trước khi rời phase.** Audit đủ 10 mục có bằng chứng.

**Khi audit FAIL.** Phase chưa hoàn thành. Codex phải ghi symptom, reproduction, expected, actual, root cause, file bị ảnh hưởng, cách sửa, regression test và residual risk vào báo cáo lỗi. Không được sang phase sau, không được tắt test, nới RLS, chuyển bucket public, thêm `any`/`@ts-ignore` hoặc catch lỗi giả thành công. Sau khi sửa phải chạy lại toàn bộ audit của phase này rồi mới đổi trạng thái sang PASS.


## Z7.5 — Tracker closure

**Thực hiện.** Mọi phase phải PASS hoặc SKIPPED-NOT-NEEDED có lý do; không TODO/FAIL.

**Audit bắt buộc trước khi rời phase.** Audit số phase plan và PHASES khớp.

**Khi audit FAIL.** Phase chưa hoàn thành. Codex phải ghi symptom, reproduction, expected, actual, root cause, file bị ảnh hưởng, cách sửa, regression test và residual risk vào báo cáo lỗi. Không được sang phase sau, không được tắt test, nới RLS, chuyển bucket public, thêm `any`/`@ts-ignore` hoặc catch lỗi giả thành công. Sau khi sửa phải chạy lại toàn bộ audit của phase này rồi mới đổi trạng thái sang PASS.


## Z7.6 — Release tag

**Thực hiện.** Commit cuối, tag v1-mvp và push chỉ khi working tree sạch + CI xanh.

**Audit bắt buộc trước khi rời phase.** Audit tag remote trỏ đúng commit deploy.

**Khi audit FAIL.** Phase chưa hoàn thành. Codex phải ghi symptom, reproduction, expected, actual, root cause, file bị ảnh hưởng, cách sửa, regression test và residual risk vào báo cáo lỗi. Không được sang phase sau, không được tắt test, nới RLS, chuyển bucket public, thêm `any`/`@ts-ignore` hoặc catch lỗi giả thành công. Sau khi sửa phải chạy lại toàn bộ audit của phase này rồi mới đổi trạng thái sang PASS.


## Z7.7 — Demo rehearsal

**Thực hiện.** Demo login, play, seek, search không dấu, like reload, playlist, upload pending và moderation trong dưới 10 phút; có localhost fallback cùng tag.

**Audit bắt buộc trước khi rời phase.** Audit rehearsal hai lần không chỉnh DB tay.

**Khi audit FAIL.** Phase chưa hoàn thành. Codex phải ghi symptom, reproduction, expected, actual, root cause, file bị ảnh hưởng, cách sửa, regression test và residual risk vào báo cáo lỗi. Không được sang phase sau, không được tắt test, nới RLS, chuyển bucket public, thêm `any`/`@ts-ignore` hoặc catch lỗi giả thành công. Sau khi sửa phải chạy lại toàn bộ audit của phase này rồi mới đổi trạng thái sang PASS.


## Z7.8 — Final 100/100 declaration

**Thực hiện.** Chỉ ghi 100/100 khi tracker không TODO/FAIL, clean-room xanh, security xanh, production smoke xanh và rubric đủ evidence.

**Audit bắt buộc trước khi rời phase.** Audit Codex đọc lại toàn plan và báo zero deviation hoặc liệt kê deviation còn lại.

**Khi audit FAIL.** Phase chưa hoàn thành. Codex phải ghi symptom, reproduction, expected, actual, root cause, file bị ảnh hưởng, cách sửa, regression test và residual risk vào báo cáo lỗi. Không được sang phase sau, không được tắt test, nới RLS, chuyển bucket public, thêm `any`/`@ts-ignore` hoặc catch lỗi giả thành công. Sau khi sửa phải chạy lại toàn bộ audit của phase này rồi mới đổi trạng thái sang PASS.


# 6. TỔNG SỐ PHASE VÀ GATE

Bản chốt này có **176 phase**. Việc chia nhỏ phase nhằm giảm blast radius, giúp Codex biết chính xác phạm vi thay đổi và bắt lỗi trước khi nó lan sang phần sau. Số lượng phase không phải bằng chứng chất lượng; evidence test mới là bằng chứng.

Các gate bắt buộc là D2.30 cho database reproducibility, S3.18 cho Auth/Storage/Upload security, M4.7 cho provider compatibility trước khi xây player sâu, M4.24 cho playback/catalog, Q5.24 cho social/search/admin, R6.28 cho operations/deploy và toàn bộ Z7 cho clean-room/security/release.

# 7. QUY TRÌNH CODEX Ở MỖI PHASE

Trước khi sửa file, Codex phải đọc phase hiện tại và xác nhận phase phụ thuộc đã PASS trong `PHASES.md`. Nó phải tìm implementation hiện có bằng `rg` trước khi tạo client/helper/store/type/route mới để tránh code trùng. Chỉ thay đổi phạm vi nhỏ nhất cần thiết cho mục tiêu phase.

Sau code, Codex chạy audit. Nếu fail, nó ở lại phase đó tới khi sửa xong và có regression test nếu phù hợp. Nếu phát hiện lỗi cũ không blocking, phải ghi issue/phase fix thay vì tiện tay trộn vào commit. Nếu lỗi security/data-integrity blocking, phải dừng ngay và ưu tiên fix trước.

# 8. WORKFLOW DATABASE MIGRATION-ONLY

Mọi thay đổi database đi qua `web/supabase/migrations/*.sql`. Không tạo `supabase/schemas/`; không duy trì `docs/sql` như schema thứ hai. Sau migration mới luôn chạy `npx supabase db reset`, `npx supabase test db`, generate types và test ứng dụng liên quan.

Trước deploy mới chạy `npx supabase migration list` và `npx supabase db push --dry-run`. Chỉ khi xác nhận đúng project và migration dự kiến mới chạy `npx supabase db push`. Không dùng `--include-seed` production. Không dùng `db reset --linked` production.

Nếu cloud bị drift do thao tác Dashboard/SQL Editor, phải capture drift thành migration rồi đưa repo trở lại source-of-truth; không duy trì thay đổi bí mật chỉ tồn tại trên cloud.

# 8A.0. GITHUB REMOTE TARGET ĐƯỢC KHÓA

GitHub account đích của project là `vinh17491` và repository canonical là `vinh17491/music` tại `https://github.com/vinh17491/music`. Repository GitHub phải PUBLIC theo yêu cầu của chủ dự án. Việc source code public không thay đổi access model của ứng dụng: app vẫn invite-only, dữ liệu người dùng và Storage vẫn private.

Codex dùng GitHub CLI. Trước auto-create/auto-push phải chạy `gh auth status --active --hostname github.com` và xác nhận active account là `vinh17491`. Nếu đã đăng nhập nhiều account thì được dùng `gh auth switch --hostname github.com --user vinh17491`. Nếu chưa authenticate `vinh17491`, dùng browser flow `gh auth login --hostname github.com --web --git-protocol https`; không yêu cầu người dùng dán token plaintext vào repo.

Nếu repo `vinh17491/music` chưa tồn tại và `origin` chưa có, Codex tự chạy từ `D:\music`:

```powershell
gh repo create vinh17491/music --public --source=. --remote=origin --push
```

GitHub CLI hỗ trợ tạo remote repo từ source local, chỉ định remote và push local commits. Nếu repo đã tồn tại nhưng origin chưa có, dùng `git remote add origin https://github.com/vinh17491/music.git` rồi fetch. Nếu origin tồn tại nhưng không trỏ tới `vinh17491/music`, phải dừng `GIT REMOTE MISMATCH`; không tự `set-url` vì có thể ghi nhầm project.

Sau đó mọi phase branch và checkpoint của plan đều auto-push lên `origin`. `main` vẫn không auto-merge.

# 8A. GIT BRANCH + REMOTE BACKUP WORKFLOW

Project áp dụng branch-isolation bắt buộc. Codex không được code trực tiếp trên `main` hoặc `master`.

Trước một phase làm thay đổi file, Codex phải `git fetch origin --prune`, kiểm tra dirty state và bảo toàn thay đổi người dùng. Nếu đang ở branch ổn định thì tạo branch mới dạng `codex/<phase>-<slug>`, sau đó `git push -u origin HEAD` trước khi bắt đầu thay đổi rủi ro. Việc push ngay branch đầu phase tạo recovery pointer trên remote.

Trong phase, Codex không được tạo commit rác cho từng dòng thay đổi. Chỉ tạo checkpoint khi có giá trị phục hồi: trước refactor/migration/security change lớn, sau một milestone độc lập đã test, hoặc khi phase FAIL sau lượng thay đổi đáng kể và cần lưu trạng thái điều tra. WIP commit phải ghi rõ `wip(<phase>): ...` và không bao giờ được coi là PASS.

Khi phase audit PASS, Codex cập nhật `PHASES.md`, review diff, commit theo convention của phase và push branch. Remote push thành công là một phần của phase evidence nếu remote đã được cấu hình.

`main` không auto-merge. Merge/PR chỉ diễn ra khi gate tương ứng PASS và CI xanh. Không force-push, không rewrite shared history, không `git reset --hard` để xóa thay đổi chưa hiểu nguồn gốc. Migration đã deploy production không được rewrite dù branch chưa merge các phase khác.

Nếu `origin` chưa tồn tại, GitHub auth lỗi hoặc push fail, Codex phải báo `GIT BACKUP BLOCKER`. Nó có thể tiếp tục thao tác chỉ khi task hiện tại an toàn và instruction cao hơn cho phép; với migration, security, deploy hoặc refactor lớn thì phải dừng cho đến khi remote backup hoạt động.

Ở cuối mỗi khối lớn, có thể tạo annotated checkpoint tag theo plan và push tag. Tag không thay thế branch/CI/review.

# 9. SECURITY INVARIANTS

RLS luôn bật cho bảng user-facing. Service role và Audius bearer token chỉ server. Bucket audio/covers private. Ownership lấy từ auth/session. Admin role không lấy từ client. Upload object path do server sinh. MIME/size client chỉ là precheck; complete xác minh object thật. Secret không log. UI redirect không thay RLS.

Bất kỳ “fix” nào bằng cách tắt RLS, public bucket, đưa service role vào Client Component, bỏ deny test, bỏ validation hoặc trả success giả đều bị xem là FAIL nghiêm trọng.

# 10. PROVIDER VÀ BẢN QUYỀN

Jamendo và Audius phải dùng API/credential hiện hành tại thời điểm code. Audius không được quay lại giả định “không key”. Nếu provider thay contract, chỉ sửa adapter và compatibility layer nếu unified Track/stream contract vẫn giữ được.

Chỉ upload nội dung có quyền sử dụng; private/phi lợi nhuận không tự động miễn bản quyền. Metadata attribution/license phải được giữ khi provider yêu cầu.

# 11. FINAL COMMAND CHAIN THAM KHẢO

```powershell
cd D:\music\web

npx supabase start
npx supabase db reset
npx supabase test db

npx supabase gen types --lang typescript --local > src\types\database.types.ts

npm ci
npx tsc --noEmit
npm run lint
npm run test -- --run
npm run build
npx playwright test

npx supabase migration list
npx supabase db push --dry-run
```

`npx supabase db push` không nằm trong command chain tự động cuối vì nó là hành động deploy có tác động cloud; chỉ chạy tại phase deploy sau khi dry-run và project-ref đã được xác nhận.

# 12. KẾT LUẬN CHỐT

Bản này thay thế toàn bộ plan 105/160/168/172 phase trước đó và là plan canonical duy nhất. Kiến trúc database đã được đơn giản hóa thành **Supabase Local + PostgreSQL + migration-only**. Các lỗi dependency và concurrency của bản cũ đã được sửa ở cấp plan: `is_admin()` được tạo trước policy reports; audit_log không chặn delete admin; quota, like, append/reorder playlist được atomic hóa; consent đi từ server reservation; hash dùng partial unique index; counter dễ stale bị loại khỏi MVP; Audius dùng credential hiện hành; compatibility stream được kiểm tra trước player sâu; player store chỉ có một đường dẫn canonical; phase luôn có audit và quy tắc fix.

Preflight/Bootstrap PASS chỉ cho phép bắt đầu implementation; nó không phải project completion. Codex phải tiếp tục phase-loop cho tới Z7.8 hoặc hard external blocker được chứng minh.

Implementation chỉ được gọi **100/100** khi evidence thực tế tại Z7 đầy đủ. Trước đó, plan có thể là 100/100 ở cấp thiết kế nhưng code vẫn phải được chứng minh bằng test.
