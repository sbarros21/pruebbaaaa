# pruebbaaaa

%% SIRHA - Modelo de Datos (MongoDB) - Diagrama ER orientado a colecciones
%% Notas:
%% - PK = clave primaria (ObjectId en MongoDB)
%% - FK = referencia a otra colección (ObjectId)
%% - UQ = índice único
%% - IDX = índice recomendado
%% - Algunos campos (arrays/JSON) representan embebidos

erDiagram
    STUDENTS {
        string _id PK
        string code UQ
        string name
        string email
        string programId FK
        string facultyId FK
        int    semester
        string academicStatus  %% semáforo: green|blue|red (cacheable)
        %% decisions/indicadores podrían embebirse si se requieren
    }

    FACULTIES {
        string _id PK
        string name
        string sigla
    }

    PROGRAMS {
        string _id PK
        string name
        string facultyId FK
    }

    COURSES {
        string _id PK
        string code UQ
        string name
        int    credits
        string facultyId FK
    }

    PROFESSORS {
        string _id PK
        string name
        string email
        string facultyId FK
    }

    PERIODS {
        string _id PK
        string name           %% ej: "2025-2"
        datetime requestWindowStart
        datetime requestWindowEnd
        boolean active IDX
    }

    GROUPS {
        string _id PK
        string courseId  FK
        string groupNumber
        string facultyId FK
        string professorId FK
        string periodId  FK
        int    capacityMax
        int    capacityCurrent IDX
        json[] scheduleSlots   %% [{day, start, end, room}]
        string campus
        %% IDX únicos sugeridos: (courseId, groupNumber, periodId)
    }

    ENROLLMENTS {
        string _id PK
        string studentId FK
        string courseId  FK
        string groupId   FK
        string periodId  FK
        string status        %% enrolled|cancelled|passed|failed
        number grade         %% opcional
        int    attemptNumber
        %% IDX sugeridos: (studentId, periodId), (groupId), (courseId, periodId)
    }

    REQUESTS {
        string _id PK
        string radicado UQ
        string studentId FK
        string facultyId FK
        string periodId  FK
        string sourceCourseId FK
        string sourceGroupId  FK
        string targetCourseId FK
        string targetGroupId  FK
        string observations
        string status          %% pending|in_review|approved|rejected
        int    priority        %% orden de llegada
        datetime createdAt IDX
        datetime updatedAt
        datetime resolvedAt
        json[] decisions       %% [{action, byUserId, role, reason, at}]
        %% IDX sugeridos: (status), (facultyId, status), (studentId)
    }

    %% Relaciones principales
    FACULTIES ||--o{ PROGRAMS  : contiene
    FACULTIES ||--o{ COURSES   : supervisa
    PROGRAMS  ||--o{ STUDENTS  : inscribe
    COURSES   ||--o{ GROUPS    : ofrece
    FACULTIES ||--o{ GROUPS    : supervisa
    PROFESSORS ||--o{ GROUPS   : dicta

    STUDENTS  ||--o{ ENROLLMENTS : tiene
    GROUPS    ||--o{ ENROLLMENTS : agrupa
    PERIODS   ||--o{ GROUPS      : programado-en
    PERIODS   ||--o{ ENROLLMENTS : durante

    STUDENTS  ||--o{ REQUESTS : crea
    FACULTIES ||--o{ REQUESTS : gestiona
    PERIODS   ||--o{ REQUESTS : durante

    %% Doble relación de REQUESTS con GROUPS y COURSES (origen/destino)
    GROUPS  ||--o{ REQUESTS : sourceGroup (sourceGroupId)
    GROUPS  ||--o{ REQUESTS : targetGroup (targetGroupId)
    COURSES ||--o{ REQUESTS : sourceCourse (sourceCourseId)
    COURSES ||--o{ REQUESTS : targetCourse (targetCourseId)
