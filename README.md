---
title: "SOP - DI TRÚ HẠ TẦNG NHAKHO-ARCHIVE SANG CLOUDFLARE PAGES"
subtitle: "Giao thức chuẩn hóa định tuyến DNS, thiết lập SSL/TLS Edge Certificates và tối ưu hóa TTFB cho hệ thống lưu trữ tĩnh"
author: "Tang Duc Tri"
role: "Archivist"
date: "2026-06-25"
category: "Infrastructure"
tags: [Cloudflare Pages, DNS Migration, SSL Edge, TTFB Optimization, Infrastructure SOP]
---

## I. KIẾN TRÚC DI TRÚ VÀ THIẾT LẬP KẾT NỐI REPOSITORY
Giao thức chuyển đổi hạ tầng nhakho-archive từ Netlify sang Cloudflare Pages được thực hiện bằng cách đấu nối trực tiếp cổng Webhook từ kho lưu trữ GitHub sang hệ thống biên (Edge Network) của Cloudflare. Cơ chế này triệt tiêu hoàn toàn độ trễ build thụ động bằng cách tự động phân phối các tệp tĩnh (HTML/CSS/JS) đến hơn 310 trung tâm dữ liệu toàn cầu ngay khi có lệnh push mã nguồn.

- **Đấu nối GitHub Webhook**: Cloudflare Pages thiết lập một ứng dụng GitHub App ủy quyền để nhận tín hiệu thay đổi mã nguồn từ nhánh `main` của repository `tangductri123/nhakho-archive`. Khi phát hiện commit mới, hệ thống tự động kéo mã nguồn về và khởi chạy quy trình deploy trực tiếp mà không cần cài đặt các máy chủ trung gian.
- **Cấu hình môi trường (Build Settings)**: Vì dự án hiện tại là một trang web tĩnh thuần túy (`index.html`, `archive.html`, `editorial.html`), cấu hình build được chuẩn hóa với lệnh Build Command để trống và thư mục đầu ra (Build Output Directory) trỏ trực tiếp về thư mục gốc (`/`).

## II. ĐỊNH VỊ DNS VÀ CẤU HÌNH BẢN GHI ĐƯỜNG TRUYỀN (CNAME ROUTING)
Chuyển đổi luồng DNS từ Netlify sang Cloudflare yêu cầu thay đổi bản ghi tên miền chính (Apex Domain) và tên miền phụ (Subdomain) sang hệ thống định tuyến proxy của Cloudflare. Quy trình này đảm bảo toàn bộ lưu lượng truy cập được lọc qua lớp bảo vệ DDoS lớp 7 và định hướng địa chỉ IP biên tối ưu cho người dùng tại Việt Nam.

- **Bản ghi Subdomain (nhakho89.netlify.app -> pages.dev)**: Thiết lập một bản ghi `CNAME` trỏ từ tên miền phụ mong muốn (ví dụ: `nhakho89.netlify.app` hoặc tên miền tùy chỉnh) về địa chỉ đích `.pages.dev` do Cloudflare cung cấp. Kích hoạt tính năng "Proxied" (đám mây màu vàng) để ẩn địa chỉ IP nguồn và áp dụng bộ lọc bảo mật Cloudflare WAF.
- **Cơ chế nén bản ghi Apex Domain (Flattening CNAME)**: Để sử dụng tên miền gốc không có `www` trỏ về Cloudflare Pages, áp dụng kỹ thuật CNAME Flattening của Cloudflare. Hệ thống tự động phân giải bản ghi `CNAME` thành các địa chỉ `A` tương ứng tại thời điểm truy vấn, đảm bảo tuân thủ tiêu chuẩn RFC mà vẫn duy trì định tuyến biên linh hoạt.

## III. TỐI ƯU HÓA SSL/TLS EDGE CERTIFICATES VÀ CHỈ SỐ TTFB
Tối ưu hóa chỉ số phản hồi đầu tiên (TTFB) và thiết lập chứng chỉ bảo mật SSL/TLS Edge Certificates trên Cloudflare Pages được thực hiện thông qua giao thức mã hóa kép và định tuyến Anycast. Giải pháp này giúp nén thời gian bắt tay mã hóa (SSL Handshake) xuống dưới 20ms và phân phối nội dung lưu trữ với độ trễ thấp nhất.

- **Giao thức mã hóa SSL/TLS Strict Mode**: Cấu hình chế độ bảo mật SSL sang mức "Strict" để bắt buộc toàn bộ dữ liệu truyền tải giữa trình duyệt người dùng, Cloudflare Edge và máy chủ lưu trữ Pages đều phải đi qua kênh mã hóa HTTPS với chứng chỉ SHA-256/ECDSA hợp lệ.
- **Kỹ thuật tối ưu hóa TTFB tại Edge**: Cloudflare Pages lưu trữ tĩnh trực tiếp toàn bộ thư mục nhakho-archive trên các ổ đĩa NVMe biên toàn cầu. Khi người dùng tại Ho Chi Minh City truy cập, yêu cầu được phân giải tức thì tại trung tâm dữ liệu gần nhất (ví dụ: VNPT/FPT Singapore hoặc CDN Việt Nam), triệt tiêu hoàn toàn thời gian phản hồi máy chủ gốc (Origin Server Response Time), đưa TTFB trung bình về mức tiệm cận < 50ms.
