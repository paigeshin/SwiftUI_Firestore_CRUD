### Dependencies

```
# Uncomment the next line to define a global platform for your project
# platform :ios, '9.0'

target 'TodolistFirebase' do
  # Comment the next line if you don't want to use dynamic frameworks
  use_frameworks!

  # Pods for TodolistFirebase

  pod 'Firebase/Core'
  pod 'Firebase/Firestore'
  pod 'FirebaseFirestoreSwift'
  pod 'Firebase/Analytics'

end
```

### CRUD

```swift
//
//  ContentView.swift
//  TodolistFirebase
//
//  Created by paige on 2022/01/22.
//

import SwiftUI
import Firebase
import FirebaseFirestore
import FirebaseFirestoreSwift // this module enables easy integration with firestore in swift (without dictionary)

struct ContentView: View {
    
    private var db: Firestore
    @State private var title: String = ""
    @State private var tasks: [Task] = []
    
    init() {
        db = Firestore.firestore()
    }
    
    private func saveTask(task: Task) {
        do {
            _ = try db.collection("tasks")
                .addDocument(from: task) { err in
                    if let err = err {
                        print(err.localizedDescription)
                    } else {
                        print("Document has been saved!")
                        fetchAllTasks()
                    }
                }
        } catch let error {
            print(error.localizedDescription)
        }
    }
    
    private func fetchAllTasks() {
        db.collection("tasks")
            .getDocuments { snapshot, error in
                if let error = error {
                    print(error.localizedDescription)
                } else {
                    if let snapshot = snapshot {
                        tasks = snapshot.documents.compactMap { doc in
                            var task = try? doc.data(as: Task.self)
                            if task != nil {
                                task!.id = doc.documentID
                            }
                            return task
                        }
                    }
                }
            }
    }
    
    private func deleteTask(at indexSet: IndexSet) {
        indexSet.forEach { index in
            let task = tasks[index]
            db.collection("tasks")
                .document(task.id!)
                .delete { error in
                    if let error = error {
                        print(error.localizedDescription)
                    } else {
                        fetchAllTasks()
                    }
                }
        }
    }
    
    var body: some View {
        NavigationView {
            VStack {
                TextField("Enter task", text: $title)
                    .textFieldStyle(RoundedBorderTextFieldStyle())
                Button("Save") {
                    let task = Task(title: title)
                    saveTask(task: task)
                }
                
                List {
                    ForEach(tasks, id: \.title) { task in
                        NavigationLink {
                            TaskDetailView(task: task)
                        } label: {
                            Text(task.title)
                        }
                    }
                    .onDelete { indexSet in
                        deleteTask(at: indexSet)
                    }
                }
                .listStyle(PlainListStyle())
                
            }
            .onAppear {
                fetchAllTasks()
            }
            .padding()
            .navigationTitle("Tasks")
        }
    }
    
}

struct TaskDetailView: View {
    
    let db: Firestore = Firestore.firestore()
    let task: Task
    @State private var title: String = ""
    
    private func updateTask() {
        db.collection("tasks")
            .document(task.id!)
            .updateData([
                "title": title
            ]) { error in
                if let error = error {
                    print(error.localizedDescription)
                }
            }
    }
    
    var body: some View {
        VStack {
            TextField(task.title, text: $title)
                .textFieldStyle(RoundedBorderTextFieldStyle())
            Button("Update") {
                updateTask()
                
            }
        }
    }
    
}

struct ContentView_Previews: PreviewProvider {
    static var previews: some View {
        ContentView()
    }
}
```
