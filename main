import requests
import re
import json
from bs4 import BeautifulSoup


def find_courses(moodle_session):
    courses_unparsed = requests.get("https://sdo.ugatu.su", cookies={"MoodleSession": moodle_session}).text
    return list(set(re.findall(r"https://sdo\.ugatu\.su/course/view\.php\?id=\d{4}", courses_unparsed)))


def get_answers(moodle_session, election_url):
    print(election_url)
    try:
        context_id_unparsed = re.findall(r"Оценка передана: .*<br /></p></div>", requests.get(election_url, cookies={"MoodleSession": moodle_session}).text)
    except:
        return
    if len(context_id_unparsed) == 0:
        return

    return context_id_unparsed[0].split("<")[0]


def check_sdo(moodle_session):
    courses_url = find_courses(moodle_session)

    answers_object = {}
    for course_url in courses_url:

        elections_request = requests.get(course_url, cookies={"MoodleSession": moodle_session}).text
        elections_soup = BeautifulSoup(elections_request, 'html.parser')

        section_id = 0
        while True:
            section = elections_soup.find(id=f"section-{section_id}")
            if section is None:
                break

            course_title = " ".join(elections_soup.title.string.replace("Курс: ", "").strip().split())
            section_name = " ".join(section.find(class_="sectionname").string.strip().split())

            for election_object in section.find_all(class_="aalink"):
                election_name = " ".join(str(election_object.find(class_="instancename")).split('>')[1].split('<')[0].strip().split())

                answers_parsed = get_answers(moodle_session, election_object["href"])

                if answers_parsed:

                    if course_title not in answers_object:
                        answers_object[course_title] = {}
                    if section_name not in answers_object[course_title]:
                        answers_object[course_title][section_name] = {}

                    answers_object[course_title][section_name][election_name] = answers_parsed

            section_id += 1

    return answers_object


result = check_sdo("UR MOODLE TOKEN")

with open('check-100.json', 'w') as file:
    json.dump(result, file, ensure_ascii=False, indent=4)
