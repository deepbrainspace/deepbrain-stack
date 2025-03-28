stateDiagram-v2
    [*] --> LandingPage : User Arrives
    
    note right of LandingPage
      Free Credit Report
      or $1 Trial ($99/month)
    end note
    
    LandingPage --> UserDetails : Enter Details
    
    UserDetails --> Firestore : Save User Information
    
    UserDetails --> EmailVerification : Verify Email
    
    EmailVerification --> Dashboard : Create Login
    
    Dashboard --> APIPull : Soft Credit Pull
    
    APIPull --> AIQuestions : Generate Personalized Questions
    
    AIQuestions --> Firestore : Save User Answers
    
    AIQuestions --> DisputeAnalysis : Analyze Credit Issues
    
    DisputeAnalysis --> AutoDispute : Identify Disputeable Items
    
    AutoDispute --> DisputeTracking : Generate Dispute Letters
    
    note right of DisputeTracking
      Track Letter Timing
      Automated Follow-ups
      OCR Response Processing
    end note
    
    DisputeTracking --> EmailNotification : Follow-up Emails
    
    EmailNotification --> UserResponseUpload : Upload Response
    
    UserResponseUpload --> AIResponseAnalysis : AI Analyze Response
    
    state if_state <<choice>>
    DisputeTracking --> if_state : 3 Month Dispute Threshold
    
    if_state --> LegalConnection : Connect to Lawyers
    
    if_state --> CreditBuilding : Continue Credit Improvement
    
    LegalConnection --> CreditBuilding : Potential Legal Resolution
    
    CreditBuilding --> [*] : Ongoing Credit Improvement