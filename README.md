# ملف واحد يبني تطبيق أندرويد (غلاف أصلي لموقعك). لا تحذف الأسطر التي تبدأ بـ #DATA في آخر الملف
# الاستخدام: Actions ثم Build Android APK ثم Run workflow ثم أدخل الحقول الثلاثة
name: Build Android APK

on:
  workflow_dispatch:
    inputs:
      app_name:
        description: 'اسم التطبيق (يظهر تحت الأيقونة)'
        required: true
        default: 'Mahmoud chat'
      app_url:
        description: 'رابط موقعك على Netlify (يبدأ بـ https)'
        required: true
        default: 'https://harmonious-cheesecake-68491d.netlify.app'
      app_id:
        description: 'معرّف الحزمة (أحرف إنجليزية صغيرة ونقطة على الأقل)'
        required: true
        default: 'com.mahmoud.chatapp'

jobs:
  build:
    runs-on: ubuntu-latest
    env:
      APP_NAME: ${{ inputs.app_name }}
      APP_URL: ${{ inputs.app_url }}
      APP_ID: ${{ inputs.app_id }}
    steps:
      - uses: actions/checkout@v4

      - name: Unpack project from this file
        run: |
          WF="${GITHUB_WORKFLOW_REF#${GITHUB_REPOSITORY}/}"
          WF="${WF%%@*}"
          grep -q '^#DATA-END' "$WF" || { echo "::error::الملف ناقص: لم يصل النسخ حتى آخره. أعد لصق الملف كاملاً ثم احفظ"; exit 1; }
          sed -n 's/^#DATA //p' "$WF" | tr -d '\n' | base64 -d > project.tar.gz || { echo "::error::محتوى الملف تالف أو ناقص. أعد النسخ واللصق كاملاً"; exit 1; }
          echo "8b811b37b7cb05916f86a91ae9f28d654b8959974503f7ce7db0e9a679be9111  project.tar.gz" | sha256sum -c - || { echo "::error::محتوى الملف تالف أو ناقص. أعد النسخ واللصق كاملاً"; exit 1; }
          tar xzf project.tar.gz
          rm project.tar.gz

      - uses: actions/setup-java@v4
        with:
          distribution: temurin
          java-version: '17'

      - uses: gradle/actions/setup-gradle@v4
        with:
          gradle-version: '8.7'

      - name: Accept Android licenses
        run: yes | "$ANDROID_HOME/cmdline-tools/latest/bin/sdkmanager" --licenses > /dev/null || true

      - name: Build APK
        run: gradle assembleDebug --no-daemon --stacktrace

      - uses: actions/upload-artifact@v4
        with:
          name: apk
          path: app/build/outputs/apk/debug/*.apk

# ===== لا تعدّل ما بعد هذا السطر =====
#DATA H4sIAImU62oC/+19CXwTxf54uSwUhKIgD3iWNSBNIU1PSqFN2zRJabRNapIWkCNsk22zNBe7m5aW
#DATA Vk6xgEABFZ8IVEC5D21BKacoyClytwg+nnIfHlzK2f/M7ibdJJu2iPp8vz/7kSaZ+c53Zr739zuT
#DATA SGIUhVtzSXEugRrNmN+f8YSDJyY6mn4Fj+dreJ/wPq73dHtEeN+YPn5IuN9f8DhICiXA9H7/fz52
#DATA syMXt6ajVjQXs2BWChkTgICHwOw2EqdsBI6RbBN8cm22XDMmDHE1WNB8zCoD4wjUzGlmhCmDxp1h
#DATA IyhXZ0lASYARs2NWI2Y1FGow0mZ2ULit/gWk24yYmMQoocazMUWqTNOrVfoMjfolhUyn1ygy1NqQ
#DATA x90BXCRhs1EZhG0kZqDEVtSCIRJEIDOhlNRuJwUBuNVgdhgxJLg/arcHB/wP8z/bgZuNf6byN0L/
#DATA IyIiPPU/MuKJ/v+F+u/UENyIBBtsFjFqNRI23CgG0m3GDSjU0GAkHyNI8AYJjhX3EUcGI7CzEMlB
#DATA zSQGFMbvyfO/+DCaL7YTNjtGUMBW/hf0Pzwm3EP/I6Ke6P9f89iIXNb6i0fmW1Ail5SEDrKMjsxF
#DATA QuU5OGgFftpmBBGiJFOXEhr7RM3/jz3AiIf92TFAA/ofEREZ7an/4eGRT/T/7+X/gY8PYNtZaBgX
#DATA k3bUgDGDDCA8psNh2Aca7MB8aI15SFR0AN1kxHJQh5mS2aw5eC4nJOdMojQi2kKSwiziXIzCrPnC
#DATA YGlGhl4pD+bE67gVIo2MdrUA/gFodiZnIxusyECWgER4tqpgQB8cIQ4PdvUQGJmFmh2gmaQIYO6C
#DATA RUgwWJge7hG851mVSpqu4KyLViJmbyk4ZgaE1LoQQfBMTRp4KxgqECC9+dDB/hDQBSHYLCTAhTcF
#DATA QykH4ZbIcOZDKMKBccewxFfbIU25g0ibgzBgMtANyJ2Nm3GqEHkJzUezGLqIsxQarRLkUxF9Pajb
#DATA 6CElT0LB/zn7TxKGMAuKW8OkjHqDZBzPwUhKPNpi/gvsf1SkV/0nOjwq4on9/yue+ETAZKddlAiA
#DATA TRQgrpBP4KByQmMFiQkB8RZWJhAAbiX7s45AIjBRlL1/WBhpMGEWlHQ5DmCBwlB7XhiwWWFsmyCB
#DATA MU7xDhIjQ0G2YcFJOp9k+/tDUysRODHUAYiVKp1Co1LoBEhYwu9EIZXJFFqtHiAZqNa8rNfqpDrF
#DATA Y6CTAduvkT4GAo1CptbI9dJMuVL9GGjS1XJlymAGjV6r0OmUqgHax8CXodbq9Cq1TpmilEl1wKw/
#DATA DrIsZbLGSWYOihzGl/kYb0IJYwFKYGIDaCZQgQuMwEY5cAIDIkdXHDzW1UikFtwAEl2TzYrVj5jB
#DATA zAlM6oIVdpQZzcbMEkESEy2EOUMFgRcgboB6lWTB7RbUHoYb9GbUYQXaQniDEjaH1ahsPDxqNtsK
#DATA klFDnsPuXLwXDOmw220ERWoosFoYJ3iDQALKzBhKUNhoSkegOTm4wSc+Cug5Ru+70IyFSe12HWxw
#DATA 6jZDNgOF54MwwdXCHc8wRpwO3I2UhRPwAmKj4bohW9xX7c4ESBpYCZYISMAGM6ZDyTx+2ALcarQV
#DATA aG05lNJqd1DMINQ4Eth/DUbiRT6mMNARlsyEWnMxUiKA5WQrRYtEMWkgMMyqBUOLSQvgBbCOWk4T
#DATA /TYNLbQ5qOI8rDDbBoQwFTcaMavrY7EDh+sQJLhNHY9bQVhIhYL0m8II9z4XgX0qIDNYzMCI06VK
#DATA lUtV3JAAucZybURh/WicUOI0aaZKlqrQeCGLD/Ox3PgwpyCwihrG0SfgUcKcLiXhScD4N4n/oLfO
#DATA h1kQGUbrN/nHRIAN5f9utUAm/ouKiX4S//2d4j8gGnTuSLLKTMsHwtgMlxdA7MDJWingHlzuh3YT
#DATA dC+w+cA+4KhZnIbnmiixyialbVQySnAMYDwOEmPEzRaxphs6ulzaRQoSkurMs9lGhBWYwChgiMDQ
#DATA +jEBXlMOEswog+N+Px4rmo/nouzqHxMXszuaJlrn8gQJ0Ok94mgVd1HeGOIZnYaWl8NML/03EmgB
#DATA mg2YBkIPq43Ccx7fBjRU/4/x0v8+EZFP6n9/K/3PxwyUjXiM7C/APRgzUiaJIDLaaHfvMGFQkvl6
#DATA 8nGsAEaEA51D+btTXQhYoxJvRymTVxgLghUzrbcSQfcU9vEOduFQOUqhEkF6ZLgoMjVaFinuJ4pE
#DATA IkXgFfyNzo+INUeLQqNNEdGGCHGEKBw0hobTfaGRWQCchYR9YBjAUUQHUPFhDD3/+6GPm/6PRPPR
#DATA MMg8tpQbxg3TxbD3T9D/qD59+nqe/8fERMU80f+/pP4PvCqaiyGcAn5cQABugbrk1AOxsyAY59kB
#DATA oMVO+eDvNGMEJQdO35bL26+CHobNCRoEgFmYFTM3CMdcJiJ44TIwKzRtSjpn8YIA2R6d9siYV/pE
#DATA gKwfarRvLA1MYreIMxjq+1pwLoHaTbiBFCfjlAW1p6DQZngT2oIZcVQsdRhxmy9MVowSZxK4V7sN
#DATA 4IYHCfwdVqMZ4+tRWPNxwma18O3OTtjycSNGiNPhqrRgwd4oHBRuFiejJBYT7dVXgGXn4ZQYHjGA
#DATA FBq3U5CKRA5qwHyCkiCBd5gpX/0ZrpKUBhvl4BNjFpBmtwxk8tmALb6ABmLZMhNhs2AyM863/zow
#DATA DRtlNTArgNSylzDrAckC/q2Bbl8Lwo25gPs6GwrX4OyFxlyM28QpOIfF3Ea1g7I7QERKYKjFC8B3
#DATA J81aKUGghWk4Z891fUxzgN2RbcYNiMGMkiTCdTMI0CigoyTiahjD1JbsBJ4PMggExvBgZA5uRc0I
#DATA bqUQjeIVfYoyTYFIkIjwiLiGgbU6qUZHQ0c2AnqgIpmGjaoHljnqQ2SpUpVKkQbvClowkgSaSAri
#DATA 3FfP8goBrHPH5yZ88UBXhwxLQOANjAwQhNQJJXeIl2Ajdsa6gUnq+tzHcK0EgsIP7v1w13YCy2fg
#DATA 4OmpxG0MLDkr9Cq1Jl2a5j6SpQEwspmEmd11khpElgQwByykjQJhD2ZE8uEhMjDoQH4oTMgYGoRE
#DATA 8zGj0goIazVgMBXCQrhHlw47mN01hgc4jnuiDNYAVp5cd0gqZg9ZOVBwWwBIyN1fCALPkunjWS1G
#DATA 5OMGTMiaebGzyq7JUsoUnNkM9Ip4HJUwJK6uLAoYDuayAtazIiCkTDjJQUPCU1bG7cBeAM+dg96D
#DATA g8Ccgzl9BMN8QASCctjrGE+6zc+yx4SSJrAO+JICrBjjooRgz+y7EA5isASx2YYaAS2FLEl7I0Ia
#DATA wwtgKw6zGUlEBN3hgTbd2B8RCJzjS9zFnma49yY4/OWYQYQEK4STQ1awbdwNk/AyMHQQWtpBKKww
#DATA XzUKYcLrCSW3WaALApytFwrEPdnAHdUHQ7uzDDNaCPVQw5wZkJkkRgwAtAd7EtIFc89RUliih+ZU
#DATA agDpNlkPEMt7Fo53BfhozMiCQc0UckgmTlcOUsj1MrVKp1Dp9CpFlkKjl6alqQd6YtEyBwLpwGHi
#DATA djM2kK4f1C3Mjfmo0cjjh4VQiJMJ6FaEISJEoAIyn48xDQJPFGBGN/ck5GgA28KVAm+j4XxYh5Ft
#DATA s5kx1IqQJpvDbHRCAtFMA4IKKCF0GliYEIoQbzfsVBfPWeEDrC7iALLHgkD5g7LPIaGnNtkAQgni
#DATA gJCp4H09oFC+ACiYQmxHCRJjNSqkvqF4DlA3OIVT3Xr2ZPBwPtMAYrBgwEAh7AzxQuN8iosRBhi4
#DATA 14E4ZRIKcoAQwzEwMgYZgOBRxjI36VE7Tj76UMZ5MuNC+DjB2DWgV1bmgrE3bUq8Wiii0AcmmOJR
#DATA zniClkDW2jEvYqkMnnXqs5SKgSLEEcLDiRIEWHaDCREqRhsw+loNgudaQWxr5Fu999rYvUC1dkde
#DATA B1rCrzrcYNOpPG5tj6Q+rOP1ihyETBzjHVHUoy2Ew6q2ZuI6E/B/RiFYR2gC8APQlbvFH0InihDP
#DATA nT+61tusWhNjT2Ummw0YXyEnnsqiNZ43jjKwn0UIZ2wGSqAWEtaswQvfBqH6eUZgTtXzJbSe8GJI
#DATA SgMGDCS9MCE9tjHS7DWxxLWLuN8r+ik2gkmVhMymxUzo4nT9Ilco3TgNwBpLBIShWRwvsOHRKPV7
#DATA LEM92vd7ZPAlki5reLga1s47CLPrPZsEiNgswZmowsuG4IWPelDBOTUTJjkHYu5WiKMDR5/mlglW
#DATA 6ImF7ALqB86A35OCHtxBUUBdBdWrqytqJlcvrZkmECFCowgpoFWbWbSYDuIICwgU60WqtspgZG6G
#DATA 6R5mBTswcnHQXfWjAHrO5xL/cFbK2P38H2Vmzes1pYCV42smPAYzVVgu6iYhH9ZMqq6qXlr9AS/S
#DATA RnD3vyIgXE/LEZmwXkio60EAsSZVL65eV1MKdaB6BbevV5h7VuOUI5D9CFkpqbv/xJUIdlEGE2bI
#DATA 02LmHI535A6QSBD3iqA4Q6FJV2rpC7YDNFIQ3Mt9p1f15IGcpUB6xzOLTQASCpJxJit11WziE7hk
#DATA hT7wBbg/16VQH7fYQBgHscGcoWHYRuJnrtk1hJmFcsdJ65rzcrJYK39Zr1TpkAQJEhUFo2afU3rf
#DATA gPOKT+tdDM/4uoWVuG+bRoSTCosdRKVe07Ds5PKRHkDZaFbRsSDDxyHhw1jfTVe2vCbkExa+QK1R
#DATA 0Z975sPJlNQEnotbgazV1QZ4euvSHaS/R1DwOxIlV4IkYecEuQYzvu7zC14Jkg9Ki42YtdDTpDCa
#DATA y8fCR1SkHBuBOI0EAfbOoY0zQyW9hQBu0IstQJe06kyNTMFe/JRJM3SZGoVzj0RIvRLuobR8buvR
#DATA 1dg77mpg5VlKueJRV+4yB4+4Zi8D4eEO3MSpUUoJT2ZAxOyDh42VIL5qbZ1OeRX4HskSDFQku3s5
#DATA j1KsWzqo8ZqATRNgNZidHX6TxhkLDRnG8XOkCBaNQRNNE2YgyVe59TmL2wxuiN1Quqs9ZxDUdmel
#DATA HsgPL1l9pG2uSBDmKNzZxGbMmkuZkAQk3J2dtB7TdAFK7GvLbguFczfo1kOYJfCkNN5qZcsL4d0k
#DATA K5X8XfWJKAYm5cfIZxJ9iK27MS+pL8KqXlO9omYqeP2YDrdAjDqpZryvYKuBMwRnIO1LYpkGkv7i
#DATA GfzMJLuIEaVQLsd8iBSdDPPYZO8M1ylfnurO1hWBfjCrgLVtjxKO2KsowXg+l3rUrZ5etodANlBx
#DATA ILzUp9Epum/T5dRpD+q7Ud7Hun1H3R9WV1ZvhFJBiwKdt0wCgjK5ZkLNpPrCcOZwwfcpjAebfYaG
#DATA kTGerOZBB4J41tHzzcWeAIoQAb34j6srwF4+qpkE8iSeGwpiZXqGWqOTqoBDTFUOSPXgEc8IxGqB
#DATA hsrziIoPN3206oGRdnCWOmNotYh9081gaiiS5JwpcQ6TWA1jroV7Ep9p9VQX/kgQEpq9hw53TDcq
#DATA RlMEKhTAKQWegSC3LG4SW2C5Chg7wRBp6KtoaFF4aL+wF/Xi10KHjYkQRYWHlwhCXNOb4jxTtboF
#DATA lTTKDKmwgoa279SZOlAWxu0M0FdXved3XtA0PbgHdYAk8ATShxuE5WYM3vwGXK078REKzDZGJMQQ
#DATA lyTYddLXGxEEA5H2KNGVNDbegFd5M4B5IDGjl3ZyVskuGlYDBtjgGBiP0eeC7Ke6qWkP5iSwG/pH
#DATA YaIc2C7CVljfmjwpB09tgQmlz+M8D/eclDU60dZnSeXuQHw5GwWvcAjdbhxYyFy3OgNPSZ6++AG0
#DATA IQ/TYaMp7+KTCCIRsWBpCtUAXao+Ta0aEMKWWBo02+uBpV7Rn3lfUfM6MHgbq5fX1VEmA08PPi6u
#DATA KXU6+4nVy4Dnb0RdBbdC2taFRd4k4ZK/rqrnyT/2pN+Xi6ZjKiudejAzejLZ80IEjQ/aJHpeD44z
#DATA nSTb6X13QqnSy9Tp6ZkqtkDga7jWjgGeEfS31dRWXuFy0ce92lXCqANdY2D2xb+tR5nKfVduBPG1
#DATA KI9Ytr6IcDUI/z7zjAVrxtdMRZwxQPUnNVPBuwYKcvAAF94M0dnktgIrvLvgKszBa/N1tVyc8yE7
#DATA JjoEoUA8VkAidYcbnG/aF1IYCN/gCwzemHtrQKkNtNDFRIucTXJFijQzTedttHPYXxATMq91tQn4
#DATA mZtsJiICGJgJYG0EdhGY3YwaMKnZDNzYUPCE9e+VOFQQn1A8DJhfgV7gPRdVaKenglvkTgU/e0zF
#DATA +T5WmA3YQSqUpG90wekheCMrapH9PAXL7fokks/GS26tnlqTL7Y7gHFy3RkUuxgoliu1GWnSwfQP
#DATA LogYWjZ+cLoyXaHXDc4AIyFlGj9Qo0gD6pml0GdIdakihHPlESxIo5Dp1JrBerl6oCpNLZVrQ7wC
#DATA fsQB/tGhGrtv+sfugPsRhohxKwnPj3jnVQyCX3aWprnucmRqlCIknyeQoydw2jRafmkyu0RYKGDm
#DATA AYoIZMoo8EDBvcOH2Ehfa7WBdI8LCqflWQxEUO9aIB4fK+E/vbSR4gICp4CKQcXzrN4wpzBmHwMN
#DATA ZpBLCeutT7FxngNeivWGgBkZYsQJhiog6MQIMB1sJeU4IWy8NEDa0HjqJY7VBifj0gV8FFvywF83
#DATA TaGXlcPqE/wAcXvrhDdrnfBunMzhDPHmgU/6+6I9H91LGoyreQKKVSBr+hipXgYs/mSnUyitmVC9
#DATA 0ZUTVn9K906ozxcwV0uZa0mclSbx3GgK8HE+iJPM3SahdxmQ55CJcwLZmEmY+AU4K5qPjfNSYxqh
#DATA N57+j0HJ4IJIeG4P0HGlALjZ1xkGMO6Y9r8gnlsFWtbyOGKBl0427oaAazoQJ66rmQVZ7en/BfUU
#DATA a38HkU0o8UdTmYnC6+z8o1Pdy4Tz36XgZ46YJVZ1JayXgIgaRNgroF4AgGUw2J5cvRa0l4LYey0S
#DATA Ec6G34urV4PmlYJ6r1U07nIKJwd5hIijV1gv7+CCHylOmSFWt4gJYHCGRnGNuo7ET1I2Qycxq/PM
#DATA hvc6mFahkvugFBwKg2Ed2LqQJ65wgwMGl6lasOiBh9dI9VqdRiFNFyHe7tRtMGo0ppjRXNI5OCVN
#DATA OoCpGevBeDmMDvR11WRfmNzuv7GYmMKP8x4VnEzEUJ33/tsfpesgfveU3D9W2envrBamk7lCriTV
#DATA KbjNWCjiFlMap+7Ooo67vPDk0m5NfCU4WvX52QrlLkupG6zXKlUD0kDYqs5AihFeCFmaQqqBALzY
#DATA XRLH1MlEzEZ5IOGWoA7AAlIIImR/lszgIODXuHVAS9NxsxmHFwjCEPg1fR4cbl+oQuyQQm5NME12
#DATA iR4PwXAgdrjI5z0Pd1w0ETIz5FKdQi/L1GhAfAwoxAOjBAm2TprMe5ONW+t03slBrNnOrwvwV4Z9
#DATA LjDRqxbs+56PyPkNEd/3Wvo/AjqevVnpa6Na+HMk8IdkhBqx8/vcYuf3ueu/lpMGf+6NHur2nTM2
#DATA 3XUeIQndj5NEiEbM/GCNmPODNQ1cAGJTDR1UTyFjeBoFD4tYUIvrh5Y6KBtzwYi5S98Y1KxW2/EG
#DATA hMatFs9Xe6/nSyTcqyE83yWpp1DPGDYh1BfAZfrn/7wO8hp/TfkxIypu1ZN22/zFN19+2UuUxVyM
#DATA AEdI3OOuj61p/a7l8R/kMlVEF0t8Vc/AFB5M4SmR/21+JNHr9x8YPQ4dPXq0yWjHuT9AJbZbc/+M
#DATA 33+IiYjp6/H9775Pfv/1L3qmZKgGPB3QBYrj08pUuQa8lsF/LZuBv9+8+8xUP78mDzPSdIqOS9f3
#DATA rNr04OHDtvM+7lrxWexXnw84tCt0x1bF/p0AsueyNe1K53Sa8/7Ry1cSDn0hnFe+qvpb0TvvNy8q
#DATA 7VK14+3jx5utWOlXW/vm7gOqZWv9xo7tXLmpy/pN0h1bIrdul/3nkPz9RQHvroeNGzblVVYFvFuZ
#DATA WVvb6t0NLTZteX7HzjM//Qx6YzZvNm3aMWrvvsmbv1z3n//4rajtXPFZ54oq2YZK6YndfmO3Bcyt
#DATA 7DX9HcmxXZIjO6U1e6XVu8fW1q797sxTcytbzV3fas8eyfE9CaD36M7MFSuSq/d898svfn61fpYS
#DATA v7G1nVZ92nHVxo6rq55bub71B9t7HT2acHRnSvWRnNXrF3191G/mWr9/7e+wZtMzG3Z327YjaOv2
#DATA btu3B22HfxO3jX1h2zZk6+YXtm1BNlX12LA78eA2yf7Pk95ZmHRsh/T0/tSru3Nq9/+vi8n3t4Pg
#DATA T0OPozQq7cOHtfC/2lrA0NraB/DPw9oWTWprm2oOww8ax/raQ80cAbW1FS3g52ZhZbW65uJa9WHi
#DATA YNPaZ2e1EY2ubFZR26S2afjodQdqn5oZWFsb2CKtLTo3JJNonvbasgNtAzvX9p3btgX/WpLalQFx
#DATA bbVMCcLA0ScPnPIf821MF/+fOzV9cSrSRzT4E2zn8k+1cfm3zbN7Wg5L1lxbffv21nsP9WeSJixZ
#DATA 56/YtPPIrinn/O9LxraZ0CMt9a3IxOHfJG4b89v90/6SrW3wSydfMotShWPvXdE/vP9eUHbEwpUD
#DATA /M/FfXWztu/Bf9+K/jAk9SZyd7Gw/28/qG8mRQVftMwuvvfzsJ9y1EPuTk9KDW2fqfAf+1vWi1PK
#DATA 73x7Vn0wMdxEZGxr0vJSUuGl/Lnb9D0qNna+KVAGvRrbb/dLAc0PEXkbZYbw8TPmjL2/9AQ52rxo
#DATA hLpD+tLEr5u39++vfmiZvrTprYlJu9vulVXNFVHlP5xFD75ysqki+V8pcUNPTH77wp10dUai36RT
#DATA SJfDD1ccK7eu1Z/+590J9xW7W3c4XbXw/anlN852z03oPfOVoyLknUMh7fqrFlRJgpCaScXSQ5WX
#DATA B2w/v7DvluGBywI7l+TPfK6D4tdBiXulNzrVlhzvOnlcVU6rX08g6sr797KHT5hR0r/yrbKtNVnT
#DATA M+/3bimPu/3m+O96qkuuDx5ftEvV4/bF+6t+bNdrU/sWoQ9/FfkZ5weda/t8Xuqs9pXFW3ZLqclb
#DATA +900R5wYuqQzGhAyVdT+QIjp4fNP7259NujOyfZhs/aXmwpV0guF7a6PjDiwve/EJX0WdStNoQLC
#DATA Zs8skb6wOG3k0mHd1NGS1OfKNu/p9NPT8Vll/TfkS5LuXG15s0d+KLJh9oYxI6rU0sJp659O/kh5
#DATA sn2gJeq3C+se9Iw6/8PPSR0OnLm0fvypYS2Rk+NnFi0o39r17M3Y5KYnp5RfTP4CnYZMem/KuHcu
#DATA +i14em17ebu8DtImd7tMBlBhLfsVlTVp3qb51vKt/rsvoR1Wt7wcefPZ9c+VDW/+asbG+9f3feWX
#DATA HIQomhcvt94R+u8/0fXc9XU/PZ28bHzqz0nD5l9evm967PC+SMyy9YKfm45a8Nm2wgtf5yQOD0aS
#DATA Os2b16mlILN5zzLh9/Hz5i8yCy4UfnF14wjH0KT05KdmNqkMaPl2kLLD+ZoVV5YpMPmIJg5Hk52d
#DATA 5s9I6dAh0O6IH6R4qy8ySDosfeIHcc8PpxZLk1YWmwqnDOx8KVaQ+vy9vkHFhw+VUk0KC/svTGrR
#DATA 9O6sMFPHbyb6z5/Ra+SPl6jU+YEXu8/u9U7T5GeWBV4SjyuWzD/Ua1HLwvFVTU2bFpd+P27ciApD
#DATA 00NrT6ZcOSmoGl86FM0dGt/159apt6iUd6dMnXhv3KQ3KeS99iN/XJ509UxV6Q7pRqH1coj02KFZ
#DATA 6jmjrn3kv2fRab92zY61ULxWvrfDFWlh4qunN7e+noqkywNKa+e1emvairId8Z8jo45HTehv2LHg
#DATA S0dBwcT4yNSOnZtlbznWPFvQ4b2DHSe89MqIhM/LtyYu69CiqEOZ4JMWRv/OipiDReXVbSNLVwV1
#DATA lF/d166r6ecFM1vuuq5+kDei2R7BkGl7yo/KlhR9N29o54vKqmvPXIq/9Pb4mJhogb3ouJ9jGDL9
#DATA 4osfFx2ODuzwxriWnZc/uNj2Xcni6H2RTSZ1LOsxdeMb1r03YuXGwBdWXHyxOv4fb0zsNbv1W2P2
#DATA d8YlN3+LzfPL/mgrcS14oqnrZXnY+MAtbwR9ECKc4RiW2qTjolnvzbpcM2LB+qRFL7XrJe+692Lf
#DATA ouMtRg3unHT3zTeD+rY8VX73nc3PLTm4++zGtmVDvixc80z0iuBzyTe3zl2qCIyceLVzqwV5TWsi
#DATA n5GcaLGom2VJ12Zv39h6dlbSYENGoa51am9N/LVxWQP6TMgYNXyOYsXSFmX2jKjUubN2VTwlzFrY
#DATA dpD0plY09akuB4uQwj1W/4hDc8Yfa/Z9kbB17Nydl+cHF46fXPZi9y0DmqDqjNeuIG1bvJA8pEf6
#DATA tCGrhyw/fiTwy5iPJ00kDqUUvTu2OPbQrzN7lo8cOD2gy0ZpdPcYQ27n6KYh86vWxFv9kpM+kd2+
#DATA g/ZGftjV/Wlh96q8wgkrVafbyF99qv2ytc9/+kuYn7zZirlHfrglHdQ8pKSlPDDwXETU6rIxRyee
#DATA uCL7x8vIiwL/0qKW5xf8al7tvyNvQnXU80Pjv9+02bhvrT2wfXyksHxPdVTPCbZOCyWFmuKN0+cF
#DATA TzrRZHVQVUVE4oK0qWskpjY7BrU7eavJqHnPhHe98e2cPRvws9PKdU3bH/ohtXQ3UlZgWReRuMz2
#DATA 8nSZdPaAFtdOSq+veS7m/LaFS/DOs/pfX52ecvDAhK/+Fbjt5BxpYbp5fzdrt8O1bbr/+73A9jdG
#DATA TUjddfrYP3LlTZbPlb4058xba2XWSJH9n0j/FVE/9ryw4d2M4d9aOlJDWvziKEWaAuwKs03/4o5L
#DATA 7W5Fv5/dYsnGDF1Kwi5c8q/a8P/cC8yKnTxJYO2qyTq24e2c7RPLg25oZg1IaJEu3te/Z8j5ZwUJ
#DATA 5aX/ls0L8z9y6PjozZnfhXfY8AKw3qOG7V3Tc+In1Cej+gXpZ1wccTapalx0es+rX/USrzy8aF2L
#DATA inWtp0y/88XBntidq2Ou7V7a/sOr/poQxNy81ZFpldI2uYpXJOfeVW/c0aWndumW6+R+/yV4+N3l
#DATA 1hMfbsZWr0k59eHrK1+bFUm8eqvjm31uhB+J7bK/WY+ESxdqh8248/2Psb0HF7Z7WXKB9F+1TXZf
#DATA erdkcvvSMZV7b8QXFDybe6xftysHs3ZMWnbrH0mzHC+v/Sb7ftobK/t9Uzivzbz4nmV5/pbhUyv2
#DATA JlXNH/T+p7tC8otaDu+25FROTZc3B+xYPi6lMiLI/OGMSeuWNxuctCJ1dYyh2eAFxFu2y+9c7zmz
#DATA c49pVMcLFc9m5b7fJk7sEAW13r1hfFai/bNfrv0ceKq5sSbpiqmsT3xWhmPG67enHjH4n9sk+/bm
#DATA iE/9vzh8d2XE5GsfDLzQb8LE5ZLu1Oqg4qfuae/ZrlT2vH59e6urg/p9817LuPMLP7n6+fIvjswo
#DATA fX56rOS3jj0+UvU4heUt3XhqQUXz2Q+GfP36rirBwfM7fpv3bEDVhdvfnIutmjzaVPj+9uQTOSe/
#DATA /SpvnN/m2Mqdz9nfu5omie83Jfr8bztlXw263nL2nTuH983p+0/8zC8XziweuPD8tt6hCuPqxLIO
#DATA X36+bu6p0Ne2PLhFXL7USvD2ysCrw2a8pr1fNCbyh+7XYwN/iejY5Zkde8YEjmni94AYGyB9uVs3
#DATA bflTX7xxrPczs+98X7CvZfLc7JgOrY6dfTpmpV4gvXdrS/EvzyZ8NC0dv9636ZivR6ruF98SzF7U
#DATA df2nP+JNT/6gl8/usXTPw8pr69GMfsqgKWEVhfK4/Js3ChNWxa888WD/t/uWKRMKjnUSHHrt6N6O
#DATA KbFLf0w83Hvxc3s67bt08Ey/olVjMV2yYd5pfd8lT52+cGLl7dn6uJTgj+afe7UkLn7BlkkHEid3
#DATA qjgwe+dDov8W8cFDUx7sNF89l18xv5nJGPBF8cSDt3PWDDzx9pKz08K2TTiz8m74wZo9hrVz7527
#DATA lt/3p5tLtz24vnLt2l0klf/Jr726Ha9QH1tc2+35Gek3t2A96P+1iFKhkq9JHjHhyQ+vPnmePE+e
#DATA J8+T58nzX3z+H85ntRIAeAAA
#DATA-END
# Mahmoud_chat# ظ…ظ„ظپ ظˆط§ط­ط¯ ظٹط¨ظ†ظٹ طھط·ط¨ظٹظ‚ ط£ظ†ط¯ط±ظˆظٹط¯ (ط؛ظ„ط§ظپ ط£طµظ„ظٹ ظ„ظ…ظˆظ‚ط¹ظƒ). ظ„ط§ طھط­ط°ظپ ط§ظ„ط£ط³ط·ط± ط§ظ„طھظٹ طھط¨ط¯ط£ ط¨ظ€ #DATA ظپظٹ ط¢ط®ط± ط§ظ„ظ…ظ„ظپ
# ط§ظ„ط§ط³طھط®ط¯ط§ظ…: Actions ط«ظ… Build Android APK ط«ظ… Run workflow ط«ظ… ط£ط¯ط®ظ„ ط§ظ„ط­ظ‚ظˆظ„ ط§ظ„ط«ظ„ط§ط«ط©
name: Build Android APK

on:
  workflow_dispatch:
    inputs:
      app_name:
        description: 'ط§ط³ظ… ط§ظ„طھط·ط¨ظٹظ‚ (ظٹط¸ظ‡ط± طھط­طھ ط§ظ„ط£ظٹظ‚ظˆظ†ط©)'
        required: true
        default: 'Mahmoud chat'
      app_url:
        description: 'ط±ط§ط¨ط· ظ…ظˆظ‚ط¹ظƒ ط¹ظ„ظ‰ Netlify (ظٹط¨ط¯ط£ ط¨ظ€ https)'
        required: true
        default: 'https://harmonious-cheesecake-68491d.netlify.app'
      app_id:
        description: 'ظ…ط¹ط±ظ‘ظپ ط§ظ„ط­ط²ظ…ط© (ط£ط­ط±ظپ ط¥ظ†ط¬ظ„ظٹط²ظٹط© طµط؛ظٹط±ط© ظˆظ†ظ‚ط·ط© ط¹ظ„ظ‰ ط§ظ„ط£ظ‚ظ„)'
        required: true
        default: 'com.mahmoud.chatapp'

jobs:
  build:
    runs-on: ubuntu-latest
    env:
      APP_NAME: ${{ inputs.app_name }}
      APP_URL: ${{ inputs.app_url }}
      APP_ID: ${{ inputs.app_id }}
    steps:
      - uses: actions/checkout@v4

      - name: Unpack project from this file
        run: |
          WF="${GITHUB_WORKFLOW_REF#${GITHUB_REPOSITORY}/}"
          WF="${WF%%@*}"
          grep -q '^#DATA-END' "$WF" || { echo "::error::ط§ظ„ظ…ظ„ظپ ظ†ط§ظ‚طµ: ظ„ظ… ظٹطµظ„ ط§ظ„ظ†ط³ط® ط­طھظ‰ ط¢ط®ط±ظ‡. ط£ط¹ط¯ ظ„طµظ‚ ط§ظ„ظ…ظ„ظپ ظƒط§ظ…ظ„ط§ظ‹ ط«ظ… ط§ط­ظپط¸"; exit 1; }
          sed -n 's/^#DATA //p' "$WF" | tr -d '\n' | base64 -d > project.tar.gz || { echo "::error::ظ…ط­طھظˆظ‰ ط§ظ„ظ…ظ„ظپ طھط§ظ„ظپ ط£ظˆ ظ†ط§ظ‚طµ. ط£ط¹ط¯ ط§ظ„ظ†ط³ط® ظˆط§ظ„ظ„طµظ‚ ظƒط§ظ…ظ„ط§ظ‹"; exit 1; }
          echo "8b811b37b7cb05916f86a91ae9f28d654b8959974503f7ce7db0e9a679be9111  project.tar.gz" | sha256sum -c - || { echo "::error::ظ…ط­طھظˆظ‰ ط§ظ„ظ…ظ„ظپ طھط§ظ„ظپ ط£ظˆ ظ†ط§ظ‚طµ. ط£ط¹ط¯ ط§ظ„ظ†ط³ط® ظˆط§ظ„ظ„طµظ‚ ظƒط§ظ…ظ„ط§ظ‹"; exit 1; }
          tar xzf project.tar.gz
          rm project.tar.gz

      - uses: actions/setup-java@v4
        with:
          distribution: temurin
          java-version: '17'

      - uses: gradle/actions/setup-gradle@v4
        with:
          gradle-version: '8.7'

      - name: Accept Android licenses
        run: yes | "$ANDROID_HOME/cmdline-tools/latest/bin/sdkmanager" --licenses > /dev/null || true

      - name: Build APK
        run: gradle assembleDebug --no-daemon --stacktrace

      - uses: actions/upload-artifact@v4
        with:
          name: apk
          path: app/build/outputs/apk/debug/*.apk

# ===== ظ„ط§ طھط¹ط¯ظ‘ظ„ ظ…ط§ ط¨ط¹ط¯ ظ‡ط°ط§ ط§ظ„ط³ط·ط± =====
#DATA H4sIAImU62oC/+19CXwTxf54uSwUhKIgD3iWNSBNIU1PSqFN2zRJabRNapIWkCNsk22zNBe7m5aW
#DATA Vk6xgEABFZ8IVEC5D21BKacoyClytwg+nnIfHlzK2f/M7ibdJJu2iPp8vz/7kSaZ+c53Zr739zuT
#DATA SGIUhVtzSXEugRrNmN+f8YSDJyY6mn4Fj+dreJ/wPq73dHtEeN+YPn5IuN9f8DhICiXA9H7/fz52
#DATA syMXt6ajVjQXs2BWChkTgICHwOw2EqdsBI6RbBN8cm22XDMmDHE1WNB8zCoD4wjUzGlmhCmDxp1h
#DATA IyhXZ0lASYARs2NWI2Y1FGow0mZ2ULit/gWk24yYmMQoocazMUWqTNOrVfoMjfolhUyn1ygy1NqQ
#DATA x90BXCRhs1EZhG0kZqDEVtSCIRJEIDOhlNRuJwUBuNVgdhgxJLg/arcHB/wP8z/bgZuNf6byN0L/
#DATA IyIiPPU/MuKJ/v+F+u/UENyIBBtsFjFqNRI23CgG0m3GDSjU0GAkHyNI8AYJjhX3EUcGI7CzEMlB
#DATA zSQGFMbvyfO/+DCaL7YTNjtGUMBW/hf0Pzwm3EP/I6Ke6P9f89iIXNb6i0fmW1Ail5SEDrKMjsxF
#DATA QuU5OGgFftpmBBGiJFOXEhr7RM3/jz3AiIf92TFAA/ofEREZ7an/4eGRT/T/7+X/gY8PYNtZaBgX
#DATA k3bUgDGDDCA8psNh2Aca7MB8aI15SFR0AN1kxHJQh5mS2aw5eC4nJOdMojQi2kKSwiziXIzCrPnC
#DATA YGlGhl4pD+bE67gVIo2MdrUA/gFodiZnIxusyECWgER4tqpgQB8cIQ4PdvUQGJmFmh2gmaQIYO6C
#DATA RUgwWJge7hG851mVSpqu4KyLViJmbyk4ZgaE1LoQQfBMTRp4KxgqECC9+dDB/hDQBSHYLCTAhTcF
#DATA QykH4ZbIcOZDKMKBccewxFfbIU25g0ibgzBgMtANyJ2Nm3GqEHkJzUezGLqIsxQarRLkUxF9Pajb
#DATA 6CElT0LB/zn7TxKGMAuKW8OkjHqDZBzPwUhKPNpi/gvsf1SkV/0nOjwq4on9/yue+ETAZKddlAiA
#DATA TRQgrpBP4KByQmMFiQkB8RZWJhAAbiX7s45AIjBRlL1/WBhpMGEWlHQ5DmCBwlB7XhiwWWFsmyCB
#DATA MU7xDhIjQ0G2YcFJOp9k+/tDUysRODHUAYiVKp1Co1LoBEhYwu9EIZXJFFqtHiAZqNa8rNfqpDrF
#DATA Y6CTAduvkT4GAo1CptbI9dJMuVL9GGjS1XJlymAGjV6r0OmUqgHax8CXodbq9Cq1TpmilEl1wKw/
#DATA DrIsZbLGSWYOihzGl/kYb0IJYwFKYGIDaCZQgQuMwEY5cAIDIkdXHDzW1UikFtwAEl2TzYrVj5jB
#DATA zAlM6oIVdpQZzcbMEkESEy2EOUMFgRcgboB6lWTB7RbUHoYb9GbUYQXaQniDEjaH1ahsPDxqNtsK
#DATA klFDnsPuXLwXDOmw220ERWoosFoYJ3iDQALKzBhKUNhoSkegOTm4wSc+Cug5Ru+70IyFSe12HWxw
#DATA 6jZDNgOF54MwwdXCHc8wRpwO3I2UhRPwAmKj4bohW9xX7c4ESBpYCZYISMAGM6ZDyTx+2ALcarQV
#DATA aG05lNJqd1DMINQ4Eth/DUbiRT6mMNARlsyEWnMxUiKA5WQrRYtEMWkgMMyqBUOLSQvgBbCOWk4T
#DATA /TYNLbQ5qOI8rDDbBoQwFTcaMavrY7EDh+sQJLhNHY9bQVhIhYL0m8II9z4XgX0qIDNYzMCI06VK
#DATA lUtV3JAAucZybURh/WicUOI0aaZKlqrQeCGLD/Ox3PgwpyCwihrG0SfgUcKcLiXhScD4N4n/oLfO
#DATA h1kQGUbrN/nHRIAN5f9utUAm/ouKiX4S//2d4j8gGnTuSLLKTMsHwtgMlxdA7MDJWingHlzuh3YT
#DATA dC+w+cA+4KhZnIbnmiixyialbVQySnAMYDwOEmPEzRaxphs6ulzaRQoSkurMs9lGhBWYwChgiMDQ
#DATA +jEBXlMOEswog+N+Px4rmo/nouzqHxMXszuaJlrn8gQJ0Ok94mgVd1HeGOIZnYaWl8NML/03EmgB
#DATA mg2YBkIPq43Ccx7fBjRU/4/x0v8+EZFP6n9/K/3PxwyUjXiM7C/APRgzUiaJIDLaaHfvMGFQkvl6
#DATA 8nGsAEaEA51D+btTXQhYoxJvRymTVxgLghUzrbcSQfcU9vEOduFQOUqhEkF6ZLgoMjVaFinuJ4pE
#DATA IkXgFfyNzo+INUeLQqNNEdGGCHGEKBw0hobTfaGRWQCchYR9YBjAUUQHUPFhDD3/+6GPm/6PRPPR
#DATA MMg8tpQbxg3TxbD3T9D/qD59+nqe/8fERMU80f+/pP4PvCqaiyGcAn5cQABugbrk1AOxsyAY59kB
#DATA oMVO+eDvNGMEJQdO35bL26+CHobNCRoEgFmYFTM3CMdcJiJ44TIwKzRtSjpn8YIA2R6d9siYV/pE
#DATA gKwfarRvLA1MYreIMxjq+1pwLoHaTbiBFCfjlAW1p6DQZngT2oIZcVQsdRhxmy9MVowSZxK4V7sN
#DATA 4IYHCfwdVqMZ4+tRWPNxwma18O3OTtjycSNGiNPhqrRgwd4oHBRuFiejJBYT7dVXgGXn4ZQYHjGA
#DATA FBq3U5CKRA5qwHyCkiCBd5gpX/0ZrpKUBhvl4BNjFpBmtwxk8tmALb6ABmLZMhNhs2AyM863/zow
#DATA DRtlNTArgNSylzDrAckC/q2Bbl8Lwo25gPs6GwrX4OyFxlyM28QpOIfF3Ea1g7I7QERKYKjFC8B3
#DATA J81aKUGghWk4Z891fUxzgN2RbcYNiMGMkiTCdTMI0CigoyTiahjD1JbsBJ4PMggExvBgZA5uRc0I
#DATA bqUQjeIVfYoyTYFIkIjwiLiGgbU6qUZHQ0c2AnqgIpmGjaoHljnqQ2SpUpVKkQbvClowkgSaSAri
#DATA 3FfP8goBrHPH5yZ88UBXhwxLQOANjAwQhNQJJXeIl2Ajdsa6gUnq+tzHcK0EgsIP7v1w13YCy2fg
#DATA 4OmpxG0MLDkr9Cq1Jl2a5j6SpQEwspmEmd11khpElgQwByykjQJhD2ZE8uEhMjDoQH4oTMgYGoRE
#DATA 8zGj0goIazVgMBXCQrhHlw47mN01hgc4jnuiDNYAVp5cd0gqZg9ZOVBwWwBIyN1fCALPkunjWS1G
#DATA 5OMGTMiaebGzyq7JUsoUnNkM9Ip4HJUwJK6uLAoYDuayAtazIiCkTDjJQUPCU1bG7cBeAM+dg96D
#DATA g8Ccgzl9BMN8QASCctjrGE+6zc+yx4SSJrAO+JICrBjjooRgz+y7EA5isASx2YYaAS2FLEl7I0Ia
#DATA wwtgKw6zGUlEBN3hgTbd2B8RCJzjS9zFnma49yY4/OWYQYQEK4STQ1awbdwNk/AyMHQQWtpBKKww
#DATA XzUKYcLrCSW3WaALApytFwrEPdnAHdUHQ7uzDDNaCPVQw5wZkJkkRgwAtAd7EtIFc89RUliih+ZU
#DATA agDpNlkPEMt7Fo53BfhozMiCQc0UckgmTlcOUsj1MrVKp1Dp9CpFlkKjl6alqQd6YtEyBwLpwGHi
#DATA djM2kK4f1C3Mjfmo0cjjh4VQiJMJ6FaEISJEoAIyn48xDQJPFGBGN/ck5GgA28KVAm+j4XxYh5Ft
#DATA s5kx1IqQJpvDbHRCAtFMA4IKKCF0GliYEIoQbzfsVBfPWeEDrC7iALLHgkD5g7LPIaGnNtkAQgni
#DATA gJCp4H09oFC+ACiYQmxHCRJjNSqkvqF4DlA3OIVT3Xr2ZPBwPtMAYrBgwEAh7AzxQuN8iosRBhi4
#DATA 14E4ZRIKcoAQwzEwMgYZgOBRxjI36VE7Tj76UMZ5MuNC+DjB2DWgV1bmgrE3bUq8Wiii0AcmmOJR
#DATA zniClkDW2jEvYqkMnnXqs5SKgSLEEcLDiRIEWHaDCREqRhsw+loNgudaQWxr5Fu999rYvUC1dkde
#DATA B1rCrzrcYNOpPG5tj6Q+rOP1ihyETBzjHVHUoy2Ew6q2ZuI6E/B/RiFYR2gC8APQlbvFH0InihDP
#DATA nT+61tusWhNjT2Ummw0YXyEnnsqiNZ43jjKwn0UIZ2wGSqAWEtaswQvfBqH6eUZgTtXzJbSe8GJI
#DATA SgMGDCS9MCE9tjHS7DWxxLWLuN8r+ik2gkmVhMymxUzo4nT9Ilco3TgNwBpLBIShWRwvsOHRKPV7
#DATA LEM92vd7ZPAlki5reLga1s47CLPrPZsEiNgswZmowsuG4IWPelDBOTUTJjkHYu5WiKMDR5/mlglW
#DATA 6ImF7ALqB86A35OCHtxBUUBdBdWrqytqJlcvrZkmECFCowgpoFWbWbSYDuIICwgU60WqtspgZG6G
#DATA 6R5mBTswcnHQXfWjAHrO5xL/cFbK2P38H2Vmzes1pYCV42smPAYzVVgu6iYhH9ZMqq6qXlr9AS/S
#DATA RnD3vyIgXE/LEZmwXkio60EAsSZVL65eV1MKdaB6BbevV5h7VuOUI5D9CFkpqbv/xJUIdlEGE2bI
#DATA 02LmHI535A6QSBD3iqA4Q6FJV2rpC7YDNFIQ3Mt9p1f15IGcpUB6xzOLTQASCpJxJit11WziE7hk
#DATA hT7wBbg/16VQH7fYQBgHscGcoWHYRuJnrtk1hJmFcsdJ65rzcrJYK39Zr1TpkAQJEhUFo2afU3rf
#DATA gPOKT+tdDM/4uoWVuG+bRoSTCosdRKVe07Ds5PKRHkDZaFbRsSDDxyHhw1jfTVe2vCbkExa+QK1R
#DATA 0Z975sPJlNQEnotbgazV1QZ4euvSHaS/R1DwOxIlV4IkYecEuQYzvu7zC14Jkg9Ki42YtdDTpDCa
#DATA y8fCR1SkHBuBOI0EAfbOoY0zQyW9hQBu0IstQJe06kyNTMFe/JRJM3SZGoVzj0RIvRLuobR8buvR
#DATA 1dg77mpg5VlKueJRV+4yB4+4Zi8D4eEO3MSpUUoJT2ZAxOyDh42VIL5qbZ1OeRX4HskSDFQku3s5
#DATA j1KsWzqo8ZqATRNgNZidHX6TxhkLDRnG8XOkCBaNQRNNE2YgyVe59TmL2wxuiN1Quqs9ZxDUdmel
#DATA HsgPL1l9pG2uSBDmKNzZxGbMmkuZkAQk3J2dtB7TdAFK7GvLbguFczfo1kOYJfCkNN5qZcsL4d0k
#DATA K5X8XfWJKAYm5cfIZxJ9iK27MS+pL8KqXlO9omYqeP2YDrdAjDqpZryvYKuBMwRnIO1LYpkGkv7i
#DATA GfzMJLuIEaVQLsd8iBSdDPPYZO8M1ylfnurO1hWBfjCrgLVtjxKO2KsowXg+l3rUrZ5etodANlBx
#DATA ILzUp9Epum/T5dRpD+q7Ud7Hun1H3R9WV1ZvhFJBiwKdt0wCgjK5ZkLNpPrCcOZwwfcpjAebfYaG
#DATA kTGerOZBB4J41tHzzcWeAIoQAb34j6srwF4+qpkE8iSeGwpiZXqGWqOTqoBDTFUOSPXgEc8IxGqB
#DATA hsrziIoPN3206oGRdnCWOmNotYh9081gaiiS5JwpcQ6TWA1jroV7Ep9p9VQX/kgQEpq9hw53TDcq
#DATA RlMEKhTAKQWegSC3LG4SW2C5Chg7wRBp6KtoaFF4aL+wF/Xi10KHjYkQRYWHlwhCXNOb4jxTtboF
#DATA lTTKDKmwgoa279SZOlAWxu0M0FdXved3XtA0PbgHdYAk8ATShxuE5WYM3vwGXK078REKzDZGJMQQ
#DATA lyTYddLXGxEEA5H2KNGVNDbegFd5M4B5IDGjl3ZyVskuGlYDBtjgGBiP0eeC7Ke6qWkP5iSwG/pH
#DATA YaIc2C7CVljfmjwpB09tgQmlz+M8D/eclDU60dZnSeXuQHw5GwWvcAjdbhxYyFy3OgNPSZ6++AG0
#DATA IQ/TYaMp7+KTCCIRsWBpCtUAXao+Ta0aEMKWWBo02+uBpV7Rn3lfUfM6MHgbq5fX1VEmA08PPi6u
#DATA KXU6+4nVy4Dnb0RdBbdC2taFRd4k4ZK/rqrnyT/2pN+Xi6ZjKiudejAzejLZ80IEjQ/aJHpeD44z
#DATA nSTb6X13QqnSy9Tp6ZkqtkDga7jWjgGeEfS31dRWXuFy0ce92lXCqANdY2D2xb+tR5nKfVduBPG1
#DATA KI9Ytr6IcDUI/z7zjAVrxtdMRZwxQPUnNVPBuwYKcvAAF94M0dnktgIrvLvgKszBa/N1tVyc8yE7
#DATA JjoEoUA8VkAidYcbnG/aF1IYCN/gCwzemHtrQKkNtNDFRIucTXJFijQzTedttHPYXxATMq91tQn4
#DATA mZtsJiICGJgJYG0EdhGY3YwaMKnZDNzYUPCE9e+VOFQQn1A8DJhfgV7gPRdVaKenglvkTgU/e0zF
#DATA +T5WmA3YQSqUpG90wekheCMrapH9PAXL7fokks/GS26tnlqTL7Y7gHFy3RkUuxgoliu1GWnSwfQP
#DATA LogYWjZ+cLoyXaHXDc4AIyFlGj9Qo0gD6pml0GdIdakihHPlESxIo5Dp1JrBerl6oCpNLZVrQ7wC
#DATA fsQB/tGhGrtv+sfugPsRhohxKwnPj3jnVQyCX3aWprnucmRqlCIknyeQoydw2jRafmkyu0RYKGDm
#DATA AYoIZMoo8EDBvcOH2Ehfa7WBdI8LCqflWQxEUO9aIB4fK+E/vbSR4gICp4CKQcXzrN4wpzBmHwMN
#DATA ZpBLCeutT7FxngNeivWGgBkZYsQJhiog6MQIMB1sJeU4IWy8NEDa0HjqJY7VBifj0gV8FFvywF83
#DATA TaGXlcPqE/wAcXvrhDdrnfBunMzhDPHmgU/6+6I9H91LGoyreQKKVSBr+hipXgYs/mSnUyitmVC9
#DATA 0ZUTVn9K906ozxcwV0uZa0mclSbx3GgK8HE+iJPM3SahdxmQ55CJcwLZmEmY+AU4K5qPjfNSYxqh
#DATA N57+j0HJ4IJIeG4P0HGlALjZ1xkGMO6Y9r8gnlsFWtbyOGKBl0427oaAazoQJ66rmQVZ7en/BfUU
#DATA a38HkU0o8UdTmYnC6+z8o1Pdy4Tz36XgZ46YJVZ1JayXgIgaRNgroF4AgGUw2J5cvRa0l4LYey0S
#DATA Ec6G34urV4PmlYJ6r1U07nIKJwd5hIijV1gv7+CCHylOmSFWt4gJYHCGRnGNuo7ET1I2Qycxq/PM
#DATA hvc6mFahkvugFBwKg2Ed2LqQJ65wgwMGl6lasOiBh9dI9VqdRiFNFyHe7tRtMGo0ppjRXNI5OCVN
#DATA OoCpGevBeDmMDvR11WRfmNzuv7GYmMKP8x4VnEzEUJ33/tsfpesgfveU3D9W2envrBamk7lCriTV
#DATA KbjNWCjiFlMap+7Ooo67vPDk0m5NfCU4WvX52QrlLkupG6zXKlUD0kDYqs5AihFeCFmaQqqBALzY
#DATA XRLH1MlEzEZ5IOGWoA7AAlIIImR/lszgIODXuHVAS9NxsxmHFwjCEPg1fR4cbl+oQuyQQm5NME12
#DATA iR4PwXAgdrjI5z0Pd1w0ETIz5FKdQi/L1GhAfAwoxAOjBAm2TprMe5ONW+t03slBrNnOrwvwV4Z9
#DATA LjDRqxbs+56PyPkNEd/3Wvo/AjqevVnpa6Na+HMk8IdkhBqx8/vcYuf3ueu/lpMGf+6NHur2nTM2
#DATA 3XUeIQndj5NEiEbM/GCNmPODNQ1cAGJTDR1UTyFjeBoFD4tYUIvrh5Y6KBtzwYi5S98Y1KxW2/EG
#DATA hMatFs9Xe6/nSyTcqyE83yWpp1DPGDYh1BfAZfrn/7wO8hp/TfkxIypu1ZN22/zFN19+2UuUxVyM
#DATA AEdI3OOuj61p/a7l8R/kMlVEF0t8Vc/AFB5M4SmR/21+JNHr9x8YPQ4dPXq0yWjHuT9AJbZbc/+M
#DATA 33+IiYjp6/H9775Pfv/1L3qmZKgGPB3QBYrj08pUuQa8lsF/LZuBv9+8+8xUP78mDzPSdIqOS9f3
#DATA rNr04OHDtvM+7lrxWexXnw84tCt0x1bF/p0AsueyNe1K53Sa8/7Ry1cSDn0hnFe+qvpb0TvvNy8q
#DATA 7VK14+3jx5utWOlXW/vm7gOqZWv9xo7tXLmpy/pN0h1bIrdul/3nkPz9RQHvroeNGzblVVYFvFuZ
#DATA WVvb6t0NLTZteX7HzjM//Qx6YzZvNm3aMWrvvsmbv1z3n//4rajtXPFZ54oq2YZK6YndfmO3Bcyt
#DATA 7DX9HcmxXZIjO6U1e6XVu8fW1q797sxTcytbzV3fas8eyfE9CaD36M7MFSuSq/d898svfn61fpYS
#DATA v7G1nVZ92nHVxo6rq55bub71B9t7HT2acHRnSvWRnNXrF3191G/mWr9/7e+wZtMzG3Z327YjaOv2
#DATA btu3B22HfxO3jX1h2zZk6+YXtm1BNlX12LA78eA2yf7Pk95ZmHRsh/T0/tSru3Nq9/+vi8n3t4Pg
#DATA T0OPozQq7cOHtfC/2lrA0NraB/DPw9oWTWprm2oOww8ax/raQ80cAbW1FS3g52ZhZbW65uJa9WHi
#DATA YNPaZ2e1EY2ubFZR26S2afjodQdqn5oZWFsb2CKtLTo3JJNonvbasgNtAzvX9p3btgX/WpLalQFx
#DATA bbVMCcLA0ScPnPIf821MF/+fOzV9cSrSRzT4E2zn8k+1cfm3zbN7Wg5L1lxbffv21nsP9WeSJixZ
#DATA 56/YtPPIrinn/O9LxraZ0CMt9a3IxOHfJG4b89v90/6SrW3wSydfMotShWPvXdE/vP9eUHbEwpUD
#DATA /M/FfXWztu/Bf9+K/jAk9SZyd7Gw/28/qG8mRQVftMwuvvfzsJ9y1EPuTk9KDW2fqfAf+1vWi1PK
#DATA 73x7Vn0wMdxEZGxr0vJSUuGl/Lnb9D0qNna+KVAGvRrbb/dLAc0PEXkbZYbw8TPmjL2/9AQ52rxo
#DATA hLpD+tLEr5u39++vfmiZvrTprYlJu9vulVXNFVHlP5xFD75ysqki+V8pcUNPTH77wp10dUai36RT
#DATA SJfDD1ccK7eu1Z/+590J9xW7W3c4XbXw/anlN852z03oPfOVoyLknUMh7fqrFlRJgpCaScXSQ5WX
#DATA B2w/v7DvluGBywI7l+TPfK6D4tdBiXulNzrVlhzvOnlcVU6rX08g6sr797KHT5hR0r/yrbKtNVnT
#DATA M+/3bimPu/3m+O96qkuuDx5ftEvV4/bF+6t+bNdrU/sWoQ9/FfkZ5weda/t8Xuqs9pXFW3ZLqclb
#DATA +900R5wYuqQzGhAyVdT+QIjp4fNP7259NujOyfZhs/aXmwpV0guF7a6PjDiwve/EJX0WdStNoQLC
#DATA Zs8skb6wOG3k0mHd1NGS1OfKNu/p9NPT8Vll/TfkS5LuXG15s0d+KLJh9oYxI6rU0sJp659O/kh5
#DATA sn2gJeq3C+se9Iw6/8PPSR0OnLm0fvypYS2Rk+NnFi0o39r17M3Y5KYnp5RfTP4CnYZMem/KuHcu
#DATA +i14em17ebu8DtImd7tMBlBhLfsVlTVp3qb51vKt/rsvoR1Wt7wcefPZ9c+VDW/+asbG+9f3feWX
#DATA HIQomhcvt94R+u8/0fXc9XU/PZ28bHzqz0nD5l9evm967PC+SMyy9YKfm45a8Nm2wgtf5yQOD0aS
#DATA Os2b16mlILN5zzLh9/Hz5i8yCy4UfnF14wjH0KT05KdmNqkMaPl2kLLD+ZoVV5YpMPmIJg5Hk52d
#DATA 5s9I6dAh0O6IH6R4qy8ySDosfeIHcc8PpxZLk1YWmwqnDOx8KVaQ+vy9vkHFhw+VUk0KC/svTGrR
#DATA 9O6sMFPHbyb6z5/Ra+SPl6jU+YEXu8/u9U7T5GeWBV4SjyuWzD/Ua1HLwvFVTU2bFpd+P27ciApD
#DATA 00NrT6ZcOSmoGl86FM0dGt/159apt6iUd6dMnXhv3KQ3KeS99iN/XJ509UxV6Q7pRqH1coj02KFZ
#DATA 6jmjrn3kv2fRab92zY61ULxWvrfDFWlh4qunN7e+noqkywNKa+e1emvairId8Z8jo45HTehv2LHg
#DATA S0dBwcT4yNSOnZtlbznWPFvQ4b2DHSe89MqIhM/LtyYu69CiqEOZ4JMWRv/OipiDReXVbSNLVwV1
#DATA lF/d166r6ecFM1vuuq5+kDei2R7BkGl7yo/KlhR9N29o54vKqmvPXIq/9Pb4mJhogb3ouJ9jGDL9
#DATA 4osfFx2ODuzwxriWnZc/uNj2Xcni6H2RTSZ1LOsxdeMb1r03YuXGwBdWXHyxOv4fb0zsNbv1W2P2
#DATA d8YlN3+LzfPL/mgrcS14oqnrZXnY+MAtbwR9ECKc4RiW2qTjolnvzbpcM2LB+qRFL7XrJe+692Lf
#DATA ouMtRg3unHT3zTeD+rY8VX73nc3PLTm4++zGtmVDvixc80z0iuBzyTe3zl2qCIyceLVzqwV5TWsi
#DATA n5GcaLGom2VJ12Zv39h6dlbSYENGoa51am9N/LVxWQP6TMgYNXyOYsXSFmX2jKjUubN2VTwlzFrY
#DATA dpD0plY09akuB4uQwj1W/4hDc8Yfa/Z9kbB17Nydl+cHF46fXPZi9y0DmqDqjNeuIG1bvJA8pEf6
#DATA tCGrhyw/fiTwy5iPJ00kDqUUvTu2OPbQrzN7lo8cOD2gy0ZpdPcYQ27n6KYh86vWxFv9kpM+kd2+
#DATA g/ZGftjV/Wlh96q8wgkrVafbyF99qv2ytc9/+kuYn7zZirlHfrglHdQ8pKSlPDDwXETU6rIxRyee
#DATA uCL7x8vIiwL/0qKW5xf8al7tvyNvQnXU80Pjv9+02bhvrT2wfXyksHxPdVTPCbZOCyWFmuKN0+cF
#DATA TzrRZHVQVUVE4oK0qWskpjY7BrU7eavJqHnPhHe98e2cPRvws9PKdU3bH/ohtXQ3UlZgWReRuMz2
#DATA 8nSZdPaAFtdOSq+veS7m/LaFS/DOs/pfX52ecvDAhK/+Fbjt5BxpYbp5fzdrt8O1bbr/+73A9jdG
#DATA TUjddfrYP3LlTZbPlb4058xba2XWSJH9n0j/FVE/9ryw4d2M4d9aOlJDWvziKEWaAuwKs03/4o5L
#DATA 7W5Fv5/dYsnGDF1Kwi5c8q/a8P/cC8yKnTxJYO2qyTq24e2c7RPLg25oZg1IaJEu3te/Z8j5ZwUJ
#DATA 5aX/ls0L8z9y6PjozZnfhXfY8AKw3qOG7V3Tc+In1Cej+gXpZ1wccTapalx0es+rX/USrzy8aF2L
#DATA inWtp0y/88XBntidq2Ou7V7a/sOr/poQxNy81ZFpldI2uYpXJOfeVW/c0aWndumW6+R+/yV4+N3l
#DATA 1hMfbsZWr0k59eHrK1+bFUm8eqvjm31uhB+J7bK/WY+ESxdqh8248/2Psb0HF7Z7WXKB9F+1TXZf
#DATA erdkcvvSMZV7b8QXFDybe6xftysHs3ZMWnbrH0mzHC+v/Sb7ftobK/t9Uzivzbz4nmV5/pbhUyv2
#DATA JlXNH/T+p7tC8otaDu+25FROTZc3B+xYPi6lMiLI/OGMSeuWNxuctCJ1dYyh2eAFxFu2y+9c7zmz
#DATA c49pVMcLFc9m5b7fJk7sEAW13r1hfFai/bNfrv0ceKq5sSbpiqmsT3xWhmPG67enHjH4n9sk+/bm
#DATA iE/9vzh8d2XE5GsfDLzQb8LE5ZLu1Oqg4qfuae/ZrlT2vH59e6urg/p9817LuPMLP7n6+fIvjswo
#DATA fX56rOS3jj0+UvU4heUt3XhqQUXz2Q+GfP36rirBwfM7fpv3bEDVhdvfnIutmjzaVPj+9uQTOSe/
#DATA /SpvnN/m2Mqdz9nfu5omie83Jfr8bztlXw263nL2nTuH983p+0/8zC8XziweuPD8tt6hCuPqxLIO
#DATA X36+bu6p0Ne2PLhFXL7USvD2ysCrw2a8pr1fNCbyh+7XYwN/iejY5Zkde8YEjmni94AYGyB9uVs3
#DATA bflTX7xxrPczs+98X7CvZfLc7JgOrY6dfTpmpV4gvXdrS/EvzyZ8NC0dv9636ZivR6ruF98SzF7U
#DATA df2nP+JNT/6gl8/usXTPw8pr69GMfsqgKWEVhfK4/Js3ChNWxa888WD/t/uWKRMKjnUSHHrt6N6O
#DATA KbFLf0w83Hvxc3s67bt08Ey/olVjMV2yYd5pfd8lT52+cGLl7dn6uJTgj+afe7UkLn7BlkkHEid3
#DATA qjgwe+dDov8W8cFDUx7sNF89l18xv5nJGPBF8cSDt3PWDDzx9pKz08K2TTiz8m74wZo9hrVz7527
#DATA lt/3p5tLtz24vnLt2l0klf/Jr726Ha9QH1tc2+35Gek3t2A96P+1iFKhkq9JHjHhyQ+vPnmePE+e
#DATA J8+T58nzX3z+H85ntRIAeAAA
#DATA-END
