"""
Pengambil komentar YouTube untuk proyek data Maluku / Pela Gandong.

Cara pakai (ringkas):
    1. pip install requests
    2. Set API key:   export YOUTUBE_API_KEY="isi_api_key"      (Mac/Linux)
                      set YOUTUBE_API_KEY=isi_api_key           (Windows CMD)
    3. Jalankan:      python youtube_scraper.py --dataset all --after 2025-01-01

Output: youtube_komentar.csv (kolom sesuai skema 10 kolom + kolom bantu).
Kolom label (konteks_konflik, sentimen, dst.) sengaja dikosongkan untuk diisi
pada tahap pelabelan.

Catatan kuota (YouTube Data API v3, gratis 10.000 unit/hari):
    - search.list          = 100 unit per panggilan
    - commentThreads.list  = 1 unit per panggilan
Skrip akan berhenti rapi bila kuota habis; data yang sudah terkumpul tetap
tersimpan dan bisa dilanjutkan besok (id duplikat otomatis dilewati).
"""

import argparse
import csv
import os
import re
import sys
import time

import requests

API_KEY = os.environ.get("YOUTUBE_API_KEY", "")
BASE = "https://www.googleapis.com/youtube/v3"

# ----------------------------------------------------------------------
# KEYWORD PER DATASET (silakan ubah / tambah)
# ----------------------------------------------------------------------
KEYWORDS = {
    "1_percakapan_konflik": [
        "konflik Ambon",
        "kerusuhan Ambon",
        "bentrok Maluku",
        "bentrokan antar desa Maluku",
        "damai Maluku",
        "Ambon manise",
        "kerukunan Maluku",
        "Muslim Kristen Ambon",
        "Malino II Maluku",
        "provokasi Ambon",
    ],
    "2_narasi_pela_gandong": [
        "Pela Gandong",
        "pela gandong Maluku",
        "basudara Maluku",
        "katong basudara",
        "Salam Sarane",
        "toleransi Maluku",
        "rekonsiliasi Ambon",
        "persaudaraan Maluku",
        "panas pela",
    ],
    "3_klarifikasi_hoaks": [
        "hoax Ambon",
        "hoaks Maluku",
        "cek fakta Maluku",
        "klarifikasi Ambon",
        "klarifikasi Maluku",
        "jangan terprovokasi Ambon",
        "provokasi hoax Maluku",
        "Polda Maluku hoaks",
    ],
}

# ----------------------------------------------------------------------
# Kolom output
# ----------------------------------------------------------------------
KOLOM_UTAMA = [
    "id",
    "tanggal",
    "platform",
    "teks",
    "konteks_konflik",
    "sentimen",
    "narasi_identitas",
    "jenis_informasi",
    "respons_ketahanan",
    "nilai_pela_gandong",
]
KOLOM_BANTU = [
    "dataset",
    "kata_kunci",
    "video_id",
    "video_judul",
    "video_url",
    "jumlah_like",
    "tipe",  # komentar / balasan
]
SEMUA_KOLOM = KOLOM_UTAMA + KOLOM_BANTU


class KuotaHabis(Exception):
    pass


def api_get(endpoint, params, retries=3):
    """Panggil YouTube API. Return dict, atau None bila gagal / dinonaktifkan."""
    params = {**params, "key": API_KEY}
    for attempt in range(retries):
        try:
            r = requests.get(f"{BASE}/{endpoint}", params=params, timeout=30)
        except requests.RequestException as e:
            print(f"   ! koneksi gagal ({e}), coba lagi...")
            time.sleep(2 ** attempt)
            continue

        if r.status_code == 200:
            return r.json()

        try:
            reason = r.json()["error"]["errors"][0]["reason"]
        except Exception:
            reason = ""

        if reason in ("quotaExceeded", "dailyLimitExceeded", "rateLimitExceeded"):
            raise KuotaHabis(reason)
        if reason in ("commentsDisabled", "videoNotFound", "forbidden"):
            return None
        if r.status_code == 400 and reason == "keyInvalid":
            print("API key tidak valid. Periksa YOUTUBE_API_KEY.")
            sys.exit(1)

        time.sleep(2 ** attempt)
    return None


def cari_video(kata_kunci, after, before, maks_video):
    """Cari video berdasarkan kata kunci. Return list (video_id, judul)."""
    hasil = []
    page_token = None
    while len(hasil) < maks_video:
        params = {
            "part": "snippet",
            "q": kata_kunci,
            "type": "video",
            "maxResults": min(50, maks_video - len(hasil)),
            "order": "relevance",
            "regionCode": "ID",
            "relevanceLanguage": "id",
        }
        if after:
            params["publishedAfter"] = after
        if before:
            params["publishedBefore"] = before
        if page_token:
            params["pageToken"] = page_token

        data = api_get("search", params)
        if not data:
            break
        for item in data.get("items", []):
            hasil.append((item["id"]["videoId"], item["snippet"]["title"]))
        page_token = data.get("nextPageToken")
        if not page_token:
            break
    return hasil


URL_RE = re.compile(r"https?://\S+|www\.\S+")
MENTION_RE = re.compile(r"@[\w.\-]+")
SPASI_RE = re.compile(r"\s+")


def bersihkan(teks):
    """Anonimisasi ringan: hapus URL & @mention, rapikan spasi."""
    teks = URL_RE.sub("", teks)
    teks = MENTION_RE.sub("", teks)
    teks = SPASI_RE.sub(" ", teks).strip()
    return teks


def ambil_komentar(video_id, maks_komentar, dengan_balasan):
    """Ambil komentar (dan balasan yang ikut terkirim) dari satu video."""
    hasil = []
    page_token = None
    while len(hasil) < maks_komentar:
        params = {
            "part": "snippet,replies" if dengan_balasan else "snippet",
            "videoId": video_id,
            "maxResults": 100,
            "order": "relevance",
            "textFormat": "plainText",
        }
        if page_token:
            params["pageToken"] = page_token

        data = api_get("commentThreads", params)
        if not data:
            break

        for thread in data.get("items", []):
            top = thread["snippet"]["topLevelComment"]
            hasil.append((top, "komentar"))
            if dengan_balasan:
                for rep in thread.get("replies", {}).get("comments", []):
                    hasil.append((rep, "balasan"))

        page_token = data.get("nextPageToken")
        if not page_token:
            break
    return hasil[:maks_komentar]


def muat_id_lama(path):
    ids = set()
    if os.path.exists(path):
        with open(path, newline="", encoding="utf-8") as f:
            for row in csv.DictReader(f):
                ids.add(row["id"])
    return ids


def main():
    ap = argparse.ArgumentParser(description="Ambil komentar YouTube (Maluku / Pela Gandong)")
    ap.add_argument("--dataset", default="all", choices=["all", "1", "2", "3"],
                    help="1=percakapan konflik, 2=Pela Gandong, 3=klarifikasi hoaks")
    ap.add_argument("--after", default="2025-01-01", help="Video terbit setelah (YYYY-MM-DD)")
    ap.add_argument("--before", default="", help="Video terbit sebelum (YYYY-MM-DD), opsional")
    ap.add_argument("--max-videos", type=int, default=10, help="Maks video per kata kunci")
    ap.add_argument("--max-comments", type=int, default=100, help="Maks komentar per video")
    ap.add_argument("--min-chars", type=int, default=10, help="Abaikan komentar lebih pendek dari ini")
    ap.add_argument("--no-replies", action="store_true", help="Jangan ambil balasan")
    ap.add_argument("--out", default="youtube_komentar.csv")
    args = ap.parse_args()

    if not API_KEY:
        print("YOUTUBE_API_KEY belum di-set. Lihat petunjuk di bagian atas file.")
        sys.exit(1)

    after = f"{args.after}T00:00:00Z" if args.after else ""
    before = f"{args.before}T00:00:00Z" if args.before else ""

    pilih = list(KEYWORDS) if args.dataset == "all" else [
        k for k in KEYWORDS if k.startswith(args.dataset + "_")
    ]

    sudah_ada = os.path.exists(args.out)
    ids_lama = muat_id_lama(args.out)
    video_selesai = set()
    total_baru = 0

    f = open(args.out, "a", newline="", encoding="utf-8-sig")
    writer = csv.DictWriter(f, fieldnames=SEMUA_KOLOM)
    if not sudah_ada:
        writer.writeheader()

    try:
        for ds in pilih:
            print(f"\n=== Dataset {ds} ===")
            for kw in KEYWORDS[ds]:
                print(f"- Kata kunci: {kw}")
                videos = cari_video(kw, after, before, args.max_videos)
                print(f"  {len(videos)} video ditemukan")

                for vid, judul in videos:
                    if vid in video_selesai:
                        continue
                    video_selesai.add(vid)

                    komentar = ambil_komentar(vid, args.max_comments, not args.no_replies)
                    baru = 0
                    for c, tipe in komentar:
                        cid = c["id"]
                        if cid in ids_lama:
                            continue
                        sn = c["snippet"]
                        teks = bersihkan(sn.get("textDisplay", ""))
                        if len(teks) < args.min_chars:
                            continue
                        ids_lama.add(cid)
                        writer.writerow({
                            "id": cid,
                            "tanggal": sn.get("publishedAt", "")[:10],
                            "platform": "YouTube",
                            "teks": teks,
                            "konteks_konflik": "",
                            "sentimen": "",
                            "narasi_identitas": "",
                            "jenis_informasi": "",
                            "respons_ketahanan": "",
                            "nilai_pela_gandong": "",
                            "dataset": ds,
                            "kata_kunci": kw,
                            "video_id": vid,
                            "video_judul": judul,
                            "video_url": f"https://www.youtube.com/watch?v={vid}",
                            "jumlah_like": sn.get("likeCount", 0),
                            "tipe": tipe,
                        })
                        baru += 1
                    f.flush()
                    total_baru += baru
                    print(f"  [{vid}] +{baru} komentar | {judul[:60]}")
    except KuotaHabis:
        print("\n! Kuota harian API habis. Data yang sudah terkumpul tersimpan; "
              "lanjutkan besok dengan perintah yang sama.")
    except KeyboardInterrupt:
        print("\nDihentikan manual. Data yang sudah terkumpul tersimpan.")
    finally:
        f.close()

    print(f"\nSelesai. {total_baru} komentar baru ditambahkan ke {args.out}")


if __name__ == "__main__":
    main()