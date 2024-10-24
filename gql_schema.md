# EduSmart Schema:

Learning Resource:

https://www.milowski.com/journal/entry/2024-06-26T12:00:00-07:00/

https://jtc1info.org/wp-content/uploads/2024/04/2024-Article-39075-Database-Language-GQL.docx.pdf

In GQL (Graph Query Language), the ISO standard syntax for comments is the same as in many other programming languages. You use two forward slashes (//) for single-line comments or /* and */ for multi-line comments.

EduSmartSchema declared via phrases:

https://shorten.wilenskid.pl/N9T.Gh
```gql
CREATE GRAPH TYPE EduSmartSchema AS {
   NODE :Program {
   program_id :: STRING NOT NULL, // PK
   name :: STRING NOT NULL,
   description :: STRING
   },


   NODE:Course {
    course_id :: STRING NOT NULL, // PK
      course_number :: STRING NOT NULL,
      name :: STRING NOT NULL,
      description :: STRING
   },

   NODE : Section {
      section_number :: STRING NOT NULL, // PK
      start_date :: DATE NOT NULL,
      end_date :: DATE NOT NULL,
      class_days_of_week :: STRING NOT NULL,
      class_time :: STRING NOT NULL,
      class_duration :: INTEGER NOT NULL,
      lab_days_of_week:: STRING NOT NULL,
      lab_time :: STRING NOT NULL,
      lab_duration :: INTEGER NOT NULL
   },

    DIRECTED EDGE INCLUDES {} CONNECTING (Program -> Course),
    DIRECTED EDGE SCHEDULED {} CONNECTING (Course -> Section),
    DIRECTED EDGE PRE_REQUISITE {} CONNECTING (Course -> Course),

   NODE :Person {
      person_id :: STRING NOT NULL, // PK
      name :: STRING NOT NULL,
      photo_url :: STRING,
      givenName :: STRING,
      familyName :: STRING NOT NULL,
      birthDate :: DATE NOT NULL,
      signupDate :: DATE NOT NULL
   },

    NODE :Student => :Person {
        student_id :: STRING NOT NULL, // pk
        roll_no :: STRING NOT NULL
    } AS Student,

    // create a separate Skill node
    NODE Profile {
        education_level :: STRING NOT NULL,       // Background education level
        work_experience :: STRING,                // Details of work experience
        english_game :: LIST<ANY>,
        iq_game :: LIST<ANY>,
        skills :: LIST<ANY>,                   // Skills based on quiz to have ["python": {}] | Can be Sep Node
        interests :: LIST<STRING>,                // Interests of the student
        preferred_learning_style :: STRING,       // E.g., visual, auditory, kinesthetic
        current_level :: STRING,                  // Current knowledge level (beginner, intermediate, advanced)
        last_updated :: DATE NOT NULL             // Last profile update date
    },

  DIRECTED EDGE HAS_PROFILE {} CONNECTING (Student -> Profile),
  DIRECTED EDGE TAKE_ADMISSION {} CONNECTING (Student -> Program),
  DIRECTED EDGE REGISTERED {
    fee_status:: STRING, 
    admission_date:: DATE 
  } CONNECTING (Student -> Section), // Suppose Fee Paid

  NODE :Teacher => :Person {
    teacher_id :: STRING NOT NULL // pk
   } AS Teacher,

   DIRECTED EDGE TEACHES {} CONNECTING (Teacher -> Section),
   DIRECTED EDGE MANAGES_ASSESSMENT {} CONNECTING (Teacher -> Assessment),
   DIRECTED EDGE MANAGES_TOPIC {} CONNECTING (Teacher -> Topic),

  NODE :TextBook {
      textbook_id :: STRING NOT NULL, // pk
      title :: STRING NOT NULL,
      author_name :: STRING NOT NULL,
      description :: STRING
   },
   DIRECTED EDGE TAUGHT {} CONNECTING (Course -> TextBook),

  NODE :Topic {
      topic_id :: STRING NOT NULL, // pk
      title :: STRING NOT NULL,
      url :: STRING,
      details :: STRING,
      topic_level :: INTEGER,
      //maximum_level :: INTEGER,
      last_reviewed :: DATE NOT NULL,
      creation_date :: DATE NOT NULL,
      sequence_number :: INT NOT NULL
   },

  DIRECTED EDGE COVERS {} CONNECTING (TextBook -> Topic),
  DIRECTED EDGE FOLLOWS CONNECTING (Topic -> Topic),

  NODE :TopicContent {
      id :: STRING NOT NULL, // pk
      content :: STRING NOT NULL, // pk
      embeddings :: LIST<ANY>,
      last_updated :: DATE NOT NULL
   },

  DIRECTED EDGE CONTAINS_CONTENT {} CONNECTING (Topic -> TopicContent),

  // This is Created after Interaction Completes (knows should change into completed)
  DIRECTED EDGE KNOWS
      {
        level :: INT NOT NULL,
        assessment_date :: DATE
      }
      CONNECTING (Student -> Topic),


//  QuestionBank Questions and Answers Related to Topics (CaseStudy is Skipped for Time)
    NODE:Question {
        question_id :: STRING NOT NULL, // pk 
        question_text :: STRING NOT NULL,
        difficulty_level :: STRING,
        points :: INT NOT NULL
    },
    
    NODE :FreeText => :Question {
      reference_answer :: STRING NOT NULL
    } AS FreeText,

    NODE :CODING => :Question {
      ideal_answer :: STRING NOT NULL
    } AS CODING,

    NODE :SingleSelect => :Question {
    } AS SingleSelect,

    NODE :MultiSelect => :Question {
    } AS MultiSelect,

    NODE :Option {
        option_id :: STRING NOT NULL, // pk
        answer_text :: STRING NOT NULL,
        is_correct :: BOOLEAN NOT NULL
    },
    
    UNDIRECTED EDGE CONTAINS {} CONNECTING (Topic ~ Question),

    DIRECTED EDGE HAS_SINGLE_OPTION {} CONNECTING (SingleSelect -> Option),
    DIRECTED EDGE HAS_MULTIPLE_OPTIONS {} CONNECTING (MultiSelect -> Option),
    DIRECTED EDGE IS_A {} CONNECTING (Question -> SingleSelect),
    DIRECTED EDGE IS_A {} CONNECTING (Question -> MultiSelect),
    DIRECTED EDGE IS_A {} CONNECTING (Question -> Coding),
    DIRECTED EDGE IS_A {} CONNECTING (Question -> FreeText),
    
// Interaction Nodes and Student Assessment Tracking

// Use-Case:
//   1. End of Course (Assessment) 2. AfterTopic/s (Interaction) 3. OnBoarding

    // 2. AfterTopic/s // (Interaction)
    NODE :Interaction {
        interaction_id :: STRING NOT NULL, // pk
        total_questions :: INT, // number of questions
        creation_date :: DATE NOT NULL,
        topic_id :: LIST<ANY>,
        end_date :: DATE
    },

    DIRECTED EDGE HAS_TOPIC {} CONNECTING (Interaction -> Topic),
    DIRECTED EDGE PARTICIPATES_IN {
        score :: INT,
        attempt_date :: DATE NOT NULL,
        percentage :: FLOAT
    } CONNECTING (Student -> Interaction),
    DIRECTED EDGE ATTEMPTED {
        score :: INT,
        answer :: STRING,
        attempt_date :: DATE NOT NULL
    } CONNECTING (Interaction -> Question),


    // 3. InitialInteraction
    NODE :InitialInteraction {
        id :: STRING NOT NULL, // pk
        title :: STRING NOT NULL,
        total_questions :: INTEGER,
        max_score :: INTEGER,
        creation_date :: DATE NOT NULL,
        questions:: LIST<ANY>,
        max_duration_minutes:: INTEGER
    },

    DIRECTED EDGE INITIATES {} CONNECTING (Student -> InitialInteraction),
    DIRECTED EDGE UPDATES {} CONNECTING (InitialInteraction -> Profile),
    DIRECTED EDGE PREREQUISITE_FOR {} CONNECTING (InitialInteraction -> Course),

    // 1. Assessment (This can be Quiz or Assessment)
    // What will we have for Final Assessment ?
    NODE : Assessment {
      id :: STRING NOT NULL, // pk
      title:: STRING ,
      details :: STRING,
      passing_score :: INTEGER
    },
    
    NODE : Assignment_Project {
      id :: STRING NOT NULL, // pk
      title :: STRING,
      details :: STRING,
      passing_score :: INTEGER,
      max_duration :: INTEGER
    },

    DIRECTED EDGE INCLUDES_PROJECT {} CONNECTING (Assessment -> Assignment_Project),
    DIRECTED EDGE HAVE_QUIZ {} CONNECTING (Assessment -> Quiz),
    DIRECTED EDGE MANDATORY_ASSESSMENT_FOR {due_date :: DATE NOT NULL} CONNECTING (Assessment -> Section),
    DIRECTED EDGE ATTEMPTS_ASSESSMENT {
      assessment_type :: STRING //midterm or final
      } CONNECTING (Student -> Assessment),

    //Edge to promote student in next course?
    NODE :Quiz {
      id :: STRING, // PK
      title:: STRING ,
      total_questions :: INTEGER,
      max_score :: INTEGER,
      time_duration :: INTEGER,
      topics:: List<STRING>
    },
    
    /* Right Now we don't have student attempted answers connected to Question. If this use case is required
    We can create an Answers nodes connected AnswerSheet to Questions.
    */
    NODE :AnswerSheet {
      id :: STRING, // PK
      question_ids :: LIST<ANY>,
      attempted_answers :: LIST<ANY>, // store [{"question": "answer", "question_id": 1 }]
      grade :: FLOAT,
      start_time :: DATE,
      end_time :: DATE
    },
    
    DIRECTED EDGE HAS_ATTEMPT {
      attempt_date :: DATE NOT NULL,
      submission_status :: STRING // ('draft', 'submitted', 'evaluated')
    } CONNECTING (Student -> AnswerSheet),

    DIRECTED EDGE ASSIGNED_QUIZ {
        assigned_date :: DATE NOT NULL,
        due_date :: DATE NOT NULL
    } CONNECTING (Student -> Quiz),
    
    DIRECTED EDGE HAS_RESPONSE {} CONNECTING (Quiz -> AnswerSheet),
    DIRECTED EDGE BELONGS_TO {} CONNECTING (Quiz -> Course),

// Classes & Tracking
    Node :OnlineSessionSchedule {
      id :: STRING NOT NULL, // pk
      zoom_link:: STRING,
      topics_to_cover:: List<STRING>,
      class_time :: DATE NOT NULL,
      class_day :: STRING,
      class_status:: BOOL
    },

    DIRECTED EDGE HAS_SECTION_SCHEDULE {} CONNECTING (Section -> OnlineSessionSchedule), //class attend
    DIRECTED EDGE HAS_COURSE_SCHEDULE {} CONNECTING (Course -> OnlineSessionSchedule), //course has course schedule

   Node :Notification {
       id :: STRING NOT NULL, // pk
      content:: STRING,
      notification_time :: DATE NOT NULL
    },

    DIRECTED EDGE NOTIFIES_STUDENT {} CONNECTING (Notification -> Student),
    DIRECTED EDGE CAN_CREATE {} CONNECTING (Teacher -> Notification),
    DIRECTED EDGE NOTIFIES_SECTION {} CONNECTING (Notification -> Section),
    DIRECTED EDGE NOTIFIES_COURSE {} CONNECTING (Notification -> Course),

    /* Purpose of this node is to have a Quick Student Course Progress Overview. Think of it as a Short Term Memory for a Student in a Particular Course in context of AGENT as teacher. */
    Node :StudentCourseStatus {
      status :: STRING, // e.g., "in progress", "completed"
      student_progress_report :: List<Any>,
      last_update :: DATE NOT NULL
    },

    DIRECTED EDGE BELONGS_TO {} CONNECTING (StudentCourseStatus -> Course),
    DIRECTED EDGE HAS_COURSE_MEMORY {} CONNECTING (Student -> StudentCourseStatus)
}

```

https://shorten.wilenskid.pl/fVVbWE

```gql
/* Insert nodes */
INSERT (:Program {name: 'Certified Generative AI Engineer', description: 'This is Agentic and Physical AI Program'})

INSERT (:Course {course_number: 'ai-101', name: 'Python AI', description: 'Coding Python using AI'})

MATCH (p : Program {name: 'Certified Generative AI Engineer'})
 ,(c : Course {course_number: 'ai-101'})
INSERT (p)-[:contains]->(c)

/* Insert two nodes and an edge */
INSERT (:Program {name: 'AI', description: 'AI Program' })
 -[:contains {}]->
 (:Course {course_number: 'ai-201', name: 'Generative AI', description: 'Coding in Generative AI'})


```
