import streamlit as st
import pandas as pd
import matplotlib.pyplot as plt
from docx import Document
from docx.shared import Inches
from docx.enum.section import WD_ORIENT
from docx.enum.text import WD_ALIGN_PARAGRAPH

# ---------------- DATA ----------------
if "students" not in st.session_state:
    st.session_state.students = [
    {
        "name": "Aaliyah Brown",
        "grades": {"Math": 78, "English": 82, "Science": 75},
        "behaviour": "B",
        "attendance": "A",
        "effort": "B",
        "comment": "A reliable student who participates well and shows steady progress across subjects."
    },
    {
        "name": "Jaden Smith",
        "grades": {"Math": 65, "English": 70, "Science": 68},
        "behaviour": "C",
        "attendance": "B",
        "effort": "C",
        "comment": "Shows moderate understanding but needs more consistency in studying and class engagement."
    },
    {
        "name": "Maya Johnson",
        "grades": {"Math": 90, "English": 88, "Science": 92},
        "behaviour": "A",
        "attendance": "A",
        "effort": "A",
        "comment": "Outstanding student with excellent academic performance and strong work ethic."
    },
    {
        "name": "Daniel Williams",
        "grades": {"Math": 55, "English": 60, "Science": 58},
        "behaviour": "C",
        "attendance": "C",
        "effort": "D",
        "comment": "Struggles academically and requires additional support and improved study habits."
    },
    {
        "name": "Chloe Davis",
        "grades": {"Math": 84, "English": 79, "Science": 81},
        "behaviour": "B",
        "attendance": "A",
        "effort": "B",
        "comment": "Strong overall performance with good class participation and consistent effort."
    },
    {
        "name": "Ethan Wilson",
        "grades": {"Math": 72, "English": 68, "Science": 74},
        "behaviour": "B",
        "attendance": "B",
        "effort": "C",
        "comment": "Demonstrates decent understanding but needs to improve focus and revision habits."
    },
    {
        "name": "Zoe Taylor",
        "grades": {"Math": 95, "English": 91, "Science": 89},
        "behaviour": "A",
        "attendance": "A",
        "effort": "A",
        "comment": "Exceptional student with top-tier academic achievement and consistent excellence."
    },
    {
        "name": "Isaac Anderson",
        "grades": {"Math": 61, "English": 65, "Science": 63},
        "behaviour": "C",
        "attendance": "B",
        "effort": "C",
        "comment": "Shows potential but needs improvement in consistency and subject understanding."
    },
    {
        "name": "Amara Thomas",
        "grades": {"Math": 88, "English": 85, "Science": 87},
        "behaviour": "A",
        "attendance": "A",
        "effort": "B",
        "comment": "Excellent student with strong academic ability and positive classroom attitude."
    },
    {
        "name": "Noah Jackson",
        "grades": {"Math": 77, "English": 73, "Science": 70},
        "behaviour": "B",
        "attendance": "B",
        "effort": "B",
        "comment": "Good performance overall with steady progress and room for improvement in focus."
    }
]

students = st.session_state.students

# ---------------- FUNCTIONS ----------------
def calculate_average(grades):
    return round(sum(grades.values()) / len(grades), 2)

def letter_grade(grade):
    if grade >= 90: return "A"
    elif grade >= 80: return "B"
    elif grade >= 70: return "C"
    elif grade >= 60: return "D"
    else: return "F"

def student_gpa(letter):
    return {"A": 4.0, "B": 3.0, "C": 2.0, "D": 1.0}.get(letter, 0.0)

def process_students():
    for s in students:
        s["average"] = calculate_average(s["grades"])
        s["letter_grades"] = {sub: letter_grade(score) for sub, score in s["grades"].items()}
        total = sum(student_gpa(l) for l in s["letter_grades"].values())
        s["GPA"] = round(total / len(s["letter_grades"]), 2)

def class_ranking():
    sorted_students = sorted(students, key=lambda x: x["average"], reverse=True)
    for i, s in enumerate(sorted_students, start=1):
        s["position"] = i

# ---------------- UI ----------------
st.set_page_config(page_title="EduTrack", layout="wide")

process_students()
class_ranking()

st.title("🎓 Student Management System")

menu = st.sidebar.selectbox("Menu", [
    "Search Student",
    "Add Student",
    "Class Dashboard",
    "Charts",
    "Export Data"
])

# ---------------- SEARCH ----------------
if menu == "Search Student":
    name = st.text_input("Enter student name")

    if st.button("Search"):
        found = False
        for s in students:
            if s["name"].lower() == name.lower():
                found = True

                st.subheader(s["name"])
                st.write(f"📊 Average: {s['average']}")
                st.write(f"🎯 GPA: {s['GPA']}")
                st.write(f"🏆 Position: {s['position']}")

                st.write("### Subjects")
                st.table(pd.DataFrame({
                    "Subject": list(s["grades"].keys()),
                    "Score": list(s["grades"].values()),
                    "Grade": list(s["letter_grades"].values())
                }))

                st.write("### Conduct")
                st.write(f"Behaviour: {s['behaviour']}")
                st.write(f"Attendance: {s['attendance']}")
                st.write(f"Effort: {s['effort']}")

                st.write("### Comment")
                st.info(s["comment"])

        if not found:
            st.error("Student not found")

# ---------------- ADD STUDENT ----------------
elif menu == "Add Student":
    st.subheader("Add New Student")

    name = st.text_input("Name")

    col1, col2, col3 = st.columns(3)
    math = col1.number_input("Math", 0, 100)
    english = col2.number_input("English", 0, 100)
    science = col3.number_input("Science", 0, 100)

    behaviour = st.selectbox("Behaviour", ["A","B","C","D","F"])
    attendance = st.selectbox("Attendance", ["A","B","C","D","F"])
    effort = st.selectbox("Effort", ["A","B","C","D","F"])

    comment = st.text_area("Teacher Comment")

    if st.button("Add Student"):
        grades = {"Math": math, "English": english, "Science": science}

        avg = calculate_average(grades)
        letters = {k: letter_grade(v) for k,v in grades.items()}
        gpa = round(sum(student_gpa(l) for l in letters.values()) / 3, 2)

        st.session_state.students.append({
            "name": name,
            "grades": grades,
            "average": avg,
            "letter_grades": letters,
            "GPA": gpa,
            "behaviour": behaviour,
            "attendance": attendance,
            "effort": effort,
            "comment": comment
        })

        st.success(f"{name} added successfully!")

# ---------------- DASHBOARD ----------------
elif menu == "Class Dashboard":
    avg_class = sum(s["average"] for s in students)/len(students)
    top = max(students, key=lambda x: x["average"])
    low = min(students, key=lambda x: x["average"])

    st.metric("Class Average", round(avg_class,2))
    st.metric("Top Student", top["name"])
    st.metric("Lowest Student", low["name"])

# ---------------- CHARTS ----------------
elif menu == "Charts":
    chart_type = st.selectbox("Choose Chart", ["Student Averages", "Subject Performance"])

    if chart_type == "Student Averages":
        names = [s["name"] for s in students]
        avgs = [s["average"] for s in students]

        fig, ax = plt.subplots()
        ax.bar(names, avgs)
        plt.xticks(rotation=45)
        st.pyplot(fig)

    else:
        subjects = ["Math", "English", "Science"]
        avgs = [sum(s["grades"][sub] for s in students)/len(students) for sub in subjects]

        fig, ax = plt.subplots()
        ax.bar(subjects, avgs)
        st.pyplot(fig)

# ---------------- EXPORT ----------------
elif menu == "Export Data":
    data = [{
        "Name": s["name"],
        "Average": s["average"],
        "GPA": s["GPA"],
        "Position": s["position"]
    } for s in students]

    df = pd.DataFrame(data)

    st.dataframe(df)

    st.download_button("Download CSV", df.to_csv(index=False), "students.csv")
