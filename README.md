# cadence-dist — kênh phân phối artifact cho Cadence selfhost

Repo này **CHỈ chứa tài liệu + artifact phát hành**. **KHÔNG chứa source code.**

- Source repo là **private** (`cuongtm2012/bot_binance_dca`) — không public, không clone từ đây.
- Artifact thật (bundle/installer) **nằm trong Releases**, KHÔNG commit vào git tree.

## 3 asset cố định của mỗi release

| Asset (tên bất biến) | Nội dung |
|---|---|
| `cadence-bundle.tar.gz` | Bundle cài đặt (core Python + control_server + units + requirements) |
| `cadence-bundle.tar.gz.sha256` | Checksum chuẩn `sha256sum` (`<hex>  cadence-bundle.tar.gz`) |
| `install-selfhost.sh` | Installer chạy trên máy client |

Tên asset **không đổi giữa các version** ⇒ URL `releases/latest/download/<asset>` là hợp đồng bất biến (SPEC §3).

## URL bất biến

```
Repo          : https://github.com/cuongtm2012/cadence-dist
Latest release: https://github.com/cuongtm2012/cadence-dist/releases/latest
Bundle        : .../releases/latest/download/cadence-bundle.tar.gz
Checksum      : .../releases/latest/download/cadence-bundle.tar.gz.sha256
Installer     : .../releases/latest/download/install-selfhost.sh
```

## Cách cài trên máy client

**Bước 1 — tải checksum + verify bundle (khuyến nghị làm trước):**

```bash
curl -fsSLO https://github.com/cuongtm2012/cadence-dist/releases/latest/download/cadence-bundle.tar.gz.sha256
curl -fsSLO https://github.com/cuongtm2012/cadence-dist/releases/latest/download/cadence-bundle.tar.gz
sha256sum -c cadence-bundle.tar.gz.sha256     # macOS: shasum -a 256 -c cadence-bundle.tar.gz.sha256
```

**Bước 2 — one-liner cài/​update (chạy bằng root trên máy client):**

```bash
curl -fsSL https://github.com/cuongtm2012/cadence-dist/releases/latest/download/install-selfhost.sh \
  | sudo bash -s -- \
      --bundle-url=https://github.com/cuongtm2012/cadence-dist/releases/latest/download/cadence-bundle.tar.gz \
      --bundle-sha256=<hex-tu-file-.sha256> \
      --engine-key=<engine_key> \
      --activation-token=<activation_token> \
      --tg-token=<bot_token> \
      --admin-chat=<chat_id>
```

Lấy `<hex-tu-file-.sha256>`:

```bash
curl -fsSL https://github.com/cuongtm2012/cadence-dist/releases/latest/download/cadence-bundle.tar.gz.sha256 | awk '{print $1}'
```

> Lưu ý: `--bundle-url` / `--bundle-sha256` cần installer build ≥ 0.1.1 (SPEC §4).
> Đường cũ `--dist-url=/--dist-token=` (control prod M1) vẫn giữ để backward-compat.

**Idempotent:** chạy lại installer = update. Nếu sha bundle khớp `.bundle_sha256` và
`cadence-engine` đang active thì installer **bỏ qua** việc tải lại.

## Rollback

Installer tự swap thư mục app và **giữ bản cũ ở `app.prev`**. Khi smoke-test sau cài fail,
installer **tự rollback** về `app.prev` và exit ≠ 0.

Rollback thủ công:

```bash
sudo systemctl stop cadence-engine cadence-control cadence-telegram 2>/dev/null || true
sudo mv /opt/cadence/app     /opt/cadence/app.broken
sudo mv /opt/cadence/app.prev /opt/cadence/app
sudo systemctl start cadence-engine cadence-control cadence-telegram
```

Rollback về một version cụ thể trên GitHub: dùng URL version hoá thay vì `latest`:

```
https://github.com/cuongtm2012/cadence-dist/releases/download/v0.1.1/cadence-bundle.tar.gz
```

## Publish release mới (maintainer)

```bash
cd /Volumes/SSD_1TB/BOT_BINANCE_core_fix/product
./scripts/release.sh 0.1.2
```

Script sẽ tạo `.sha256`, stage 3 asset tên chuẩn, `gh release create v0.1.2 ...`,
rồi in one-liner deploy cho client. Tag đã tồn tại ⇒ script thoát ≠ 0, **không ghi đè mù**.

## Cảnh báo

Bundle chứa **Python source đọc được**. Xem [`DISCLAIMER.md`](DISCLAIMER.md) trước khi publish.
