# angular_share_data_between_components

## From parent to child

```
@Component({
  selector: 'app-parent',
  standalone: true,
  imports: [ChildComponent],
  templateUrl: '
  <div>
      <h1>Parent Component</h1>
      <app-child [dataFromParent]="dataFromParent"> </app-child>
  </div>
  '
})

export class ParentComponent {
  protected dataFromParent: string = "I'm a Data from parent and displayed in child"
}


@Component({
    selector: 'app-child',
    standalone: true,
    imports: [],
    templateUrl: '
<div>
  <h2>Child</h2>
  <p>{{ dataFromParent }}</p>'
})
export class ChildComponent implements OnInit {
    // option 1
    @Input() dataFromParent: string = ''

    // option 2 using lowercase input()
    dataFromParent = input()
```

## From child to Parent
### Event emitter version

```
@Component({
    selector: 'app-child',
    standalone: true,
    imports: [],
    templateUrl: '
  <div >
    <h2>Child</h2>
    <button (click)="handleClick()">Send data to parent</button>
  </div>
'
})
export class ChildComponent implements OnInit {

    @Output() dataFromChildEvent = new EventEmitter<string>()

    handleClick() {
        this.dataFromChildEvent.emit("I'm a Data from child and displayed in parent")
    }
}



@Component({
    selector: 'app-parent',
    standalone: true,
    imports: [ChildComponent],
    templateUrl: '
  <div>
      <h1>Parent Component</h1>
      <p>{{ dataFromChild }}</p>

      <app-child (dataFromChildEvent)="handleDataFromChild($event)"> </app-child>
  </div>
'
})
export class ParentComponent implements AfterViewInit {

    public dataFromChild: string = ''

    handleDataFromChild($event: string) {
        this.dataFromChild = $event
    }
}
```


### @ViewChild version

```
@Component({
    selector: 'app-child',
    standalone: true,
    imports: [],
    templateUrl: './child.component.html'
})
export class ChildComponent implements OnInit {
    private childData = "I'm a Data of child accessed by parent through @ViewChild"
}




@Component({
    selector: 'app-parent',
    standalone: true,
    imports: [],
    templateUrl: '
    <div style="background-color: burlywood">
      <h1>Parent Component</h1>
    
      <h2>Data accessed trough child view</h2>
      <p>{{ dataAccessedByParentThroughViewChild }}</p>
    </div>

'
})
export class ParentComponent implements AfterViewInit {
    protected dataAccessedByParentThroughViewChild: string | undefined;

    @ViewChild(ChildComponent) child: any;

    ngAfterViewInit(): void {
        this.dataAccessedByParentThroughViewChild = this.child.childData
    }
}

```



## From a Service to any component 
### Behavior Subject version

```
export type BehaviorUser = {
    username: string,
    isLogged: boolean
}

@Injectable({
    providedIn: 'root'
})

export class AuthBehaviorService {
    private userSetter = new BehaviorSubject<BehaviorUser>({ username: '', isLogged: false })
    public userGetter = this.userSetter.asObservable()

    login(incomingUsername: string) {
        this.userSetter.next({ username: incomingUsername, isLogged: true })
    }

    logout() {
        this.userSetter.next({ username: '', isLogged: false })
    }
}


@Component({
    selector: 'app-child-sibling',
    standalone: true,
    imports: [],
    templateUrl: '
    <div style="background-color: cadetblue">
        <h2>Child sibling</h2>
        <button (click)="login()">Bs Login</button>
        <button (click)="logout()">Bs Logout</button>

         <div>
            <h3>Data from Behavior Subject</h3>
        
            <p>User : {{ userBS.username }}</p>
            <p>
              {{ userBS.isLogged ? "Logged In" : "Logged Out" }}
            </p>
        </div>
    </div>
'
})
export class ChildSiblingComponent implements OnInit {

    private authBehaviorService = inject(AuthBehaviorService)
    public userBS!: BehaviorUser;

    ngOnInit(): void {
        this.authBehaviorService.userGetter.subscribe(user => this.userBS = user)
    }

    login() {
        this.authBehaviorService.login("My Username")
    }

    logout() {
        this.authBehaviorService.logout()
    }

}
```

### Signal version

```
export type User = {
 name: string | undefined,
isAuthenticated: boolean
}

@Injectable({
    providedIn: 'root'
})

export class AuthContextService {
    private name = signal('')
    private isAuthenticated = signal(false)

    setUsername(incomingName: string) {
        this.name.set(incomingName)
    }
    getUsername() {
        return this.name()
    }

    setUserIsAuthenticated(incomingBoolean: boolean) {
        return this.isAuthenticated.set(incomingBoolean)
    }
    getUserIsAuthenticated() {
        return this.isAuthenticated()
    }
}



@Component({
    selector: 'app-parent',
    standalone: true,
    imports: [],
    templateUrl: '
  <div>
    <h1>Parent Component</h1>
  
    <button (click)="login()">log user in</button>
    <button (click)="logout()">log user out</button>

   <div>
      <h3>Data from Signal</h3>
      <p>User : {{ authContext.getUsername() }}</p>
      <p>
        {{ authContext.getUserIsAuthenticated() ? "Logged In" : "Logged Out" }}
      </p>
    </div>
  </div>
'
})
export class ParentComponent implements AfterViewInit {
   
    public authContext = inject(AuthContextService)

    login() {
        this.authContext.setUserIsAuthenticated(true)
        this.authContext.setUsername("Nas")
    }

    logout() {
        this.authContext.setUserIsAuthenticated(false)
        this.authContext.setUsername("")
    }
}

```


