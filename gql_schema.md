# EduSmart Schema: 

Learning Resource:

https://www.milowski.com/journal/entry/2024-06-26T12:00:00-07:00/

https://jtc1info.org/wp-content/uploads/2024/04/2024-Article-39075-Database-Language-GQL.docx.pdf

EduSmartSchema declared via phrases:

https://shorten.wilenskid.pl/M4unFR

```gql
CREATE GRAPH TYPE EduSmartSchema AS {
   NODE :Program {
      name :: STRING NOT NULL,
      description :: STRING
   },
   
   
   NODE :Course {
      course_number :: STRING NOT NULL,
      name :: STRING NOT NULL,
      description :: STRING
   },
   
   NODE : Section {
      section_number :: STRING NOT NULL,
      start_date :: DATE NOT NULL,
      end_date :: DATE NOT NULL,
      class_days_of_week :: STRING NOT NULL,
      class_time :: STRING NOT NULL,
      class_duration :: INTEGER NOT NULL,
      lab_days_of_week:: STRING NOT NULL,
      lab_time :: STRING NOT NULL,
      lab_duration :: INTEGER NOT NULL
   },
    
    DIRECTED EDGE contains {} CONNECTING (Program -> Course),
    DIRECTED EDGE scheduled {} CONNECTING (Course -> Section),
    DIRECTED EDGE pre_requisite {} CONNECTING (Course -> Course),

   NODE :Person {
      name :: STRING NOT NULL,
      url :: STRING,
      givenName :: STRING,
      familyName :: STRING NOT NULL,
      birthDate :: DATE NOT NULL,
      signupDate :: DATE NOT NULL
   },
       
    NODE :Student => :Person {
        student_id :: STRING NOT NULL 
    } AS Student,
    
    NODE :Profile {
        education_level :: STRING NOT NULL,       -- Background education level
        work_experience :: STRING,                -- Details of work experience
        english_game :: LIST<ANY>,
        iq_game :: LIST<ANY>,
        skills :: LIST<ANY>,                   -- List of skill based on quiz to have in dict ["python": {}]

        interests :: LIST<STRING>,                -- Interests of the student
        preferred_learning_style :: STRING,       -- E.g., visual, auditory, kinesthetic
        current_level :: STRING,                  -- Current knowledge level (beginner, intermediate, advanced)
        last_updated :: DATE NOT NULL             -- Last profile update date
    },
    
  DIRECTED EDGE has_profile {} CONNECTING (Student -> Profile),   
  DIRECTED EDGE take_admission {} CONNECTING (Student -> Program),
  DIRECTED EDGE registered {"status":: STRING} CONNECTING (Student -> Section), -- Suppose Fee Paid
  DIRECTED EDGE is_learning {} CONNECTING (Student -> Course), -- Supposition Fee Paid

  NODE :Teacher => :Person {
    teacher_id :: STRING NOT NULL
   } AS Teacher,

   DIRECTED EDGE teaches {} CONNECTING (Teacher -> Section),
   DIRECTED EDGE manages {} CONNECTING (Teacher -> Assessment),
   DIRECTED EDGE crud {} CONNECTING (Teacher -> Topic),

  NODE :TextBook {
      title :: STRING NOT NULL,
      author_name :: STRING NOT NULL,
      description :: STRING
   },
   DIRECTED EDGE taught {} CONNECTING (Course -> TextBook),

  NODE :Topic {
      title :: STRING NOT NULL,
      url :: STRING,
      details :: STRING,
      topic_level :: INTEGER,
      //maximum_level :: INTEGER,
      lastReviewed :: STRING NOT NULL,
      creationDate :: DATE NOT NULL,
      sequenceNumber :: INT NOT NULL
   },

  DIRECTED EDGE contains {} CONNECTING (TextBook -> Topic), 
  DIRECTED EDGE contains CONNECTING (Topic -> Topic), 
    
  // This is Created after Interaction Completes (knows should change into completed)
  DIRECTED EDGE knows 
      { 
        level :: INT NOT NULL, 
        assesment_date :: DATE
      } 
      CONNECTING (Student -> Topic),


//  QuestionBank Questions and Answers Related to Topics (CaseStudy is Skipped for Time)
    NODE:Question {
        question_text :: STRING NOT NULL,
        difficulty_level :: STRING,
        points :: INT NOT NULL
    },
    
    NODE :Answer {
        answer_text :: STRING NOT NULL,
        is_correct :: BOOLEAN NOT NULL
    },
    
    NODE :SingleSelect => :Question {
    } AS SingleSelect,
    
    NODE :MultiSelect => :Question {
    } AS MultiSelect,
    
    NODE :FreeText => :Question {
    } AS FreeText,
    
    NODE :CODING => :Question {
    } AS CODING,

    UNDIRECTED EDGE contains {} CONNECTING (Topic ~ Question), 
    DIRECTED EDGE has_answer {} CONNECTING (Question -> Answer), 
    UNDIRECTED EDGE is_a {} CONNECTING (Question ~ SingleSelect),
    UNDIRECTED EDGE is_a {} CONNECTING (Question ~ MultiSelect),
    UNDIRECTED EDGE is_a {} CONNECTING (Question ~ CODING),
    UNDIRECTED EDGE is_a {} CONNECTING (Question ~ FreeText),

// Interaction Nodes and Student Assessment Tracking

// Use-Case: 
//   1. End of Course (Assessment) 2. AfterTopic/s (Interaction) 3. OnBoarding
    
    // 2. AfterTopic/s // (Interaction)
    NODE :Interaction {
        total_questions :: INT, // number of questions
        creation_date :: DATE NOT NULL,
        topics :: LIST<STRING>,
        end_date :: DATE
    },
    
    DIRECTED EDGE interacts {
        score :: INT,
        attempt_date :: DATE NOT NULL,
        percentage :: FLOAT
    } CONNECTING (Student -> Interaction),
    DIRECTED EDGE event_for {} CONNECTING (Interaction -> Topic),
    DIRECTED EDGE attempted {
        score :: INT,
        answer :: STRING, 
        attempt_date :: DATE NOT NULL
    } CONNECTING (Interaction -> Question),
  

    // 3. InitialInteraction
    NODE :InitialInteraction {
        title :: STRING NOT NULL,
        total_questions :: INTEGER,
        max_score :: INTEGER,
        creation_date :: DATE NOT NULL,
        questions:: LIST<ANY>,
        max_duration_minutes:: INTEGER
    },
    
    DIRECTED EDGE interacts {} CONNECTING (Student -> InitialInteraction), 
    DIRECTED EDGE updates {} CONNECTING (InitialInteraction -> Profile),
    DIRECTED EDGE required_for {} CONNECTING (InitialInteraction -> Course),

    // 1. Assessment (This can be Quiz or Assessment)
    // What will we have for Final Assessment ?
    NODE : Assessment {
      title:: STRING ,
      details :: STRING,
      passing_score :: INTEGER
      
    },
    
    NODE : Assessment_Project {
      title :: STRING, 
      details :: STRING,
      passing_score :: INTEGER, 
      max_duration :: INTEGER
    },
    
    DIRECTED EDGE has_project {} CONNECTING (Assessment -> Assessment_Project), 
    DIRECTED EDGE has_quiz {} CONNECTING (Assessment -> Quiz),
    DIRECTED EDGE required_for {due_date :: DATE NOT NULL} CONNECTING (Assessment -> Section),
    DIRECTED EDGE attempts {
      assessment_type :: STRING //midterm or final
      } CONNECTING (Student -> Assessment),
    
    //Edge to promote student in next course?
    
    NODE :Quiz {
      title:: STRING ,
      total_questions :: INTEGER,
      max_score :: INTEGER,
      time_duration :: INTEGER,
      topics:: List<STRING>
    },
    
    DIRECTED EDGE attempts {} CONNECTING (Student -> Quiz), 
    DIRECTED EDGE required_for {} CONNECTING (Quiz -> Course), 

// Classes & Tracking 
    Node :OnlineSessionSchedule {
      zoom_link:: STRING,
      topics_to_cover:: List<STRING>,
      class_time :: DATE NOT NULL,
      class_day :: STRING,
      class_status:: BOOL
    },
    
    DIRECTED EDGE has {} CONNECTING (Section -> OnlineSessionSchedule), //class attend
    DIRECTED EDGE has {} CONNECTING (Course -> OnlineSessionSchedule), //course has course schedule 
   
   Node :Notification {
      content:: STRING,
      notification_time :: DATE NOT NULL
    },
    
    DIRECTED EDGE notify {} CONNECTING (Notification -> Student),
    DIRECTED EDGE can_create {} CONNECTING (Teacher -> Notification),
    DIRECTED EDGE notify {} CONNECTING (Notification -> Section),
    DIRECTED EDGE notify {} CONNECTING (Notification -> Course),
    
    Node :StudentCourseStatus {
      status :: STRING, // e.g., "in progress", "completed"
      student_progress_report :: List<Any>,
      last_update :: DATE NOT NULL
    },
    
    DIRECTED EDGE belongs_to {} CONNECTING (StudentCourseStatus -> Course),
    DIRECTED EDGE has_course_memory {} CONNECTING (Student -> StudentCourseStatus)
}

```
