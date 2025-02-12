# Simple-To-do-Project

import Map "mo:base/HashMap";
import Hash "mo:base/Hash";
import Nat "mo:base/Nat";
import Iter "mo:base/Iter";
import Text "mo:base/Text";

// Define the actor as canister
actor Assistant {

  type ToDo = {
    description: Text;
    completed: Bool;
  }; // do not miss ; 

   // default: private
   // func name(param : type) : return type
  func natHash(n : Nat) : Hash.Hash { 
    Text.hash(Nat.toText(n)) // return ... ;
  };

  var todos = Map.HashMap<Nat, ToDo>(0, Nat.equal, natHash); // let -> immutable, var -> mutable
  var nextId : Nat = 0;

  public query func getTodos() : async [ToDo] {
    Iter.toArray(todos.vals());
  };

  public func addTodo(description : Text) : async Nat {
    let id = nextId;
    todos.put(id, { description = description; completed = false });
    nextId += 1;
    id // return id;
  };
  
  public func completeTodo(id : Nat) : async () {
    ignore do ? {
      let description = todos.get(id)!.description;
      todos.put(id, { description; completed = true });
    }
  };

  public query func showTodos() : async Text {
    var output : Text = "\n___TO-DOs___";
    for (todo : ToDo in todos.vals()) {
      output #= "\n" # todo.description;
      if (todo.completed) { output #= " ✔"; };
    };
    output # "\n"
  };

  public func clearCompleted() : async () {
    todos := Map.mapFilter<Nat, ToDo, ToDo>(todos, Nat.equal, natHash, 
              func(_, todo) { if (todo.completed) null else ?todo });
  };
}
