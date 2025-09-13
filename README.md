import streamlit as st
import pandas as pd
import altair as alt
import os

st.title("MBTI 유형별 국가 비율 Top 10")

# 기본 CSV 파일 이름
default_file = "mbti_data.csv"

df = None

# 1. 기본 파일이 같은 폴더에 있는 경우 불러오기
if os.path.exists(default_file):
    df = pd.read_csv(default_file)
    st.success(f"기본 데이터 파일 **{default_file}** 로드 완료 ✅")

# 2. 기본 파일이 없는 경우 업로드 요청
else:
    uploaded_file = st.file_uploader("CSV 파일을 업로드하세요", type=["csv"])
    if uploaded_file is not None:
        df = pd.read_csv(uploaded_file)
        st.success("업로드한 CSV 파일 로드 완료 ✅")

# 데이터가 준비된 경우 실행
if df is not None:
    st.write("데이터 미리보기")
    st.dataframe(df.head())

    # MBTI와 Country 컬럼 확인
    if "MBTI" in df.columns and "Country" in df.columns:
        # MBTI별 국가 분포 계산
        mbti_counts = (
            df.groupby(["Country", "MBTI"])
            .size()
            .reset_index(name="Count")
        )

        # 비율 계산
        mbti_counts["Total"] = mbti_counts.groupby("Country")["Count"].transform("sum")
        mbti_counts["Ratio"] = mbti_counts["Count"] / mbti_counts["Total"]

        # MBTI 선택
        selected_mbti = st.selectbox("MBTI 유형을 선택하세요", sorted(df["MBTI"].unique()))

        # 선택된 MBTI 데이터만 추출
        selected_data = mbti_counts[mbti_counts["MBTI"] == selected_mbti]

        # 비율 상위 10개 국가
        top10 = selected_data.sort_values("Ratio", ascending=False).head(10)

        st.subheader(f"{selected_mbti} 비율이 가장 높은 국가 Top 10")

        # Altair 그래프
        chart = (
            alt.Chart(top10)
            .mark_bar()
            .encode(
                x=alt.X("Ratio:Q", title="비율"),
                y=alt.Y("Country:N", sort="-x", title="국가"),
                tooltip=["Country", "Ratio"]
            )
            .interactive()
        )

        st.altair_chart(chart, use_container_width=True)

    else:
        st.error("CSV 파일에 'MBTI'와 'Country' 컬럼이 있어야 합니다.")
**
