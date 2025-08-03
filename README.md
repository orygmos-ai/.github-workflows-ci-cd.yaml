# .github-workflows-ci-cd.yaml

name: CI/CD Flutter & Release APK

on:
  push:
    branches: [ main ]

jobs:
  build-release-apk:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout repository
        uses: actions/checkout@v3

      - name: Set up Flutter
        uses: subosito/flutter-action@v2
        with:
          flutter-version: '3.10.0'

      - name: Install dependencies
        run: flutter pub get

      - name: Build release APK
        run: flutter build apk --release --build-name=${{ github.run_number }} --build-number=${{ github.run_number }}

      - name: Create GitHub Release
        id: create_release
        uses: actions/create-release@v1
        with:
          tag_name: v${{ github.run_number }}
          release_name: Release ${{ github.run_number }}
          draft: false
          prerelease: false
          body: |
            APK generato automaticamente per Dispensa Sinfonica, build numero ${{ github.run_number }}.
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}

      - name: Upload APK to Release
        uses: actions/upload-release-asset@v1
        with:
          upload_url: ${{ steps.create_release.outputs.upload_url }}
          asset_path: build/app/outputs/flutter-apk/app-release.apk
          asset_name: dispensa_sinfonica_v${{ github.run_number }}.apk
          asset_content_type: application/vnd.android.package-archive
