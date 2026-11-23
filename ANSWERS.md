# Lab 6 — UML, Part 2: Sequence Diagrams (Course Registration)

## Exercise 1

* **(a)** Messages **3**, **5**, and **7** are returns.
* **(b)**
  * `Course` must have: `removeStudent(student)` (message **2**) and `getCredits()` (message **4**).
  * `UndergraduateStudent` must have: `removeCredits(6)` (message **6**).
* **(c)** As drawn, the service still calls `getCredits()` (4) and `removeCredits(6)` (6), so the student's credits are still reduced by 6 even though they were never enrolled. You should add an **`opt`** fragment (e.g., `opt student was enrolled`), with messages **4, 5, and 6** inside it.

---

## Exercise 2

```mermaid
sequenceDiagram
    participant S as service: EnrollmentService
    participant P as printer: Printer
    create participant E as enrollment: Enrollment
    S->>E: new Enrollment(student, course)
    S-)P: printConfirmation(enrollment)
```

* **(a)** Because `enrollment` is created dynamically during the interaction rather than existing at the start, so its lifeline begins at the `new Enrollment(student, course)` message.
* **(b)** In UML, an asynchronous message is drawn with a solid line and an open (stick) arrowhead.

---

## Exercise 3

```mermaid
sequenceDiagram
    participant S as service: EnrollmentService
    participant C as course: Course
    participant L as lab: LabSession
    S->>C: hasLab()
    C-->>S: has lab?
    opt course has a lab
        S->>L: reserveBench(student)
    end
```

---

## Exercise 4

```mermaid
sequenceDiagram
    actor User
    participant S as service: EnrollmentService
    participant C as course: Course
    participant W as waitlist: Waitlist
    User->>S: enroll(student, course)
    S->>C: hasSeatLeft()
    C-->>S: seat left?
    alt seat is left
        S->>C: addStudent(student)
        S-->>User: "enrolled"
    else no seat left
        S->>W: add(student)
        S-->>User: "waitlisted"
    end
```

---

## Exercise 5

```mermaid
sequenceDiagram
    participant T as transcript: Transcript
    participant E as enrollment: Enrollment
    loop for each enrollment
        T->>E: getGrade()
        E-->>T: grade
        T->>E: getCredits()
        E-->>T: credits
    end
```

---

## Exercise 6

```mermaid
sequenceDiagram
    actor User
    participant S as service: EnrollmentService
    participant C as course: Course
    participant St as student: UndergraduateStudent
    User->>S: enroll(student, course)
    S->>C: getPrerequisite()
    C-->>S: prereq
    S->>St: hasPassed(prereq)
    St-->>S: passed?
    break student has not passed prereq
        S-->>User: "missing prerequisite"
    end
    S->>C: isOpen()
    C-->>S: open?
```

---

## Exercise 7

```mermaid
sequenceDiagram
    participant G as grades: GradeService
    participant St as student: Student
    participant A as advisor: Advisor
    par notify student
        G-)St: notify(grade)
    and notify advisor
        G-)A: notify(grade)
    end
```

---

## Exercise 8

|Sentence from the requirements|Fragment|
|---|---|
|"For each course in the student's list, …"|`loop`|
|"If the student asked for a printed timetable, also print it."|`opt`|
|"The student pays the fees either by card or by bank transfer."|`alt`|
|"If registration is closed, refuse and stop."|`break`|
|"Email the student and the advisor at the same time."|`par`|

---

## Exercise 9

```mermaid
sequenceDiagram
    actor Registrar
    participant S as service: WaitlistService
    participant C as course: Course
    participant W as waitlist: Waitlist
    Registrar->>S: promote(course, waitlist)
    S->>C: hasSeatLeft()
    C-->>S: seat left?
    break no seat left
        S-->>Registrar: undefined
    end
    S->>W: next()
    W-->>S: next
    break next is undefined
        S-->>Registrar: undefined
    end
    S->>C: addStudent(next)
    S-->>Registrar: next
```

---

## Exercise 10

```typescript
enrollAll(student: Student, courses: Course[]): number {
  let acceptedCount = 0;
  for (const course of courses) {
    if (course.isOpen()) {
      const credits = course.getCredits();
      if (student.addCredits(credits)) {
        acceptedCount++;
      }
    }
  }
  return acceptedCount;
}
```

---

## Exercise 11

1. Mistakes table:

|\#|Line of the diagram|What is wrong|Correction|
|---|---|---|---|
|1|`S->>E: new Enrollment(student, course)`|Creates `enrollment` before checking if the course is open.|Move `create participant E as enrollment: Enrollment` and `S->>E: new Enrollment(student, course)` inside the `course is open` branch.|
|2|`C->>S: true`|Uses a solid call arrow (`->>`) instead of a dashed return arrow (`-->>`), and gives a single fixed answer `true`.|Change to `C-->>S: open?`.|
|3|`S->>C: seats = seats - 1`|Uses an internal attribute assignment instead of a method call on `Course`.|Change to `S->>C: addStudent(student)`.|

2. Corrected diagram:

```mermaid
sequenceDiagram
    actor User
    participant S as service: EnrollmentService
    participant C as course: Course
    User->>S: enroll(student, course)
    S->>C: isOpen()
    C-->>S: open?
    alt course is open
        S->>C: addStudent(student)
        create participant E as enrollment: Enrollment
        S->>E: new Enrollment(student, course)
        S-->>User: enrollment
    else course is closed
        S-->>User: undefined
    end
```
