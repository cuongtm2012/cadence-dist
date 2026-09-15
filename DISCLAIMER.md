# DISCLAIMER — Cadence selfhost distribution

## 1. Repo này PUBLIC và bundle chứa source Python

Repo `cuongtm2012/cadence-dist` là **PUBLIC**. Asset `cadence-bundle.tar.gz` chứa
**source Python dạng `.py` đọc được** (core engine + control_server + units).

Hệ quả: **bất kỳ ai tải được bundle đều đọc được logic bot** — không có bảo vệ source.

## 2. Đây là đánh đổi ĐÃ ĐƯỢC CHẤP NHẬN

Đánh đổi này đã được **chủ sản phẩm (Jack) chấp nhận có chủ đích** (SPEC
`docs/specs/2026-09-15-github-release-distribution.md` §2), vì:

- Máy client **không cần credential** để tải (repo public) ⇒ không phải phát token/push-key
  của repo source private cho từng khách.
- Artifact **version hoá + bất biến + verify sha256 được**, không phụ thuộc control plane còn sống.

Giảm thiểu trong tương lai (KHÔNG thuộc scope hiện tại): ship `.pyc` thay `.py`.

## 3. Release PHẢI qua review gate trước khi publish

**KHÔNG publish release tự động.** Trước mỗi `release.sh <version>`:

1. Bundle build từ source repo private (không phải từ bản nháp trên máy client).
2. Review xác nhận bundle **không chứa secret**: token Telegram, API key Binance,
   private key, `config_real.json` thật, `.env`, key license.
3. Review xác nhận bundle **không chứa file ngoài layout SPEC §9**
   (`whitelist_refresh.py`, `cvd_shadow.py`, `shadow_*`, `desktop_app/**`, `tests/**`, `docs/**`).
4. Review xác nhận `.sha256` khớp bundle vừa build.
5. Chỉ sau đó mới tạo tag/release.

Người publish chịu trách nhiệm: một release đã public là **không thu hồi được** —
người khác đã tải/copy trước khi bạn xoá.

## 4. Không đưa secret vào repo này

Không commit bất kỳ key/token/credential nào vào repo này — kể cả trong README,
issue, release notes, hay comment. Artifact chỉ nằm trong Releases, không trong git tree.

## 5. Trách nhiệm pháp lý / vận hành

Người cài đặt chịu trách nhiệm về việc dùng bot để giao dịch. Phần mềm cung cấp
"as-is"; rủi ro thị trường, rủi ro sàn, và rủi ro cấu hình thuộc về người vận hành.
