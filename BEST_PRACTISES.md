Let's take the below example of a basic handler.

```
exports.lambdaHandler = async (event, context) => {
  const { queytStringParameters } = event;
  const item = JSON.stringify(create(queytStringParameters.itemName));
  return {
    statusCode: 200,
    body: `item: ${item}`
  }
};
```

- in this, if I mistakenly type out `statusCode` as `statusCodes`, then it errors out.
- If we don't receive a `itemName` as part of `queytStringParameters` in the event, it errors out.

Now, if we Typescript-ify it as below
```
import { 
  APIGatewayProxyEvent, 
  APIGatewayProxyResult 
} from "aws-lambda";

export class QueryStringParamDTO {
  readonly itemName: string;
  ...
}

@Injectable()
export class ItemsService {
  constructor(@InjectModel('Item') private readonly itemModel: Model<Item>) {}
  async create(item: Item): Promise<Item> {
    const newItem = new this.itemModel(item);
    return await newItem.save();
  }
}

export const lambdaHandler = async (
  event: APIGatewayProxyEvent
): Promise<APIGatewayProxyResult> => {
  const { queytStringParameters: QueryStringParamDTO } = event;
  const appContext = await NestFactory.createApplicationContext(AppModule);
  const itemsService = appContext.get(ItemsService);
  
  const item = itemsService.create(queryStringParameters.itemName);
  
  return {
    statusCode: 200,
    body: `item: ${JSON.stringify(item)}`
  }
}
```

This gives more confidence of strict type checking, easier debugging and separation of concern.
Also, adding a test like below during CI gives more confidence in the system.
```
import { APIGatewayProxyEvent } from "aws-lambda";
import { lambdaHandler } from "../../src-ts/app";

describe('Unit test for app handler', function () {
    it('verifies successful response', async () => {
        const event: APIGatewayProxyEvent = {
            queryStringParameters: {
                itemName: "Item 1"
            }
        } as any
        const result = await lambdaHandler(event)

        expect(result.statusCode).toEqual(200);
        expect(result.body).toEqual(`Item: ${JSON.stringify(event.queryStringParameters.itemName)}`);
    });
});
```
